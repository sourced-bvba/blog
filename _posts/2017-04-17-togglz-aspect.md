---
layout: post
category : article
title: "Togglz aspect with Spring Boot"
comments: true
tags : [java]
---

Despite what you might think when reading my last articles, this boy still writes code. Today I'm looking at [Togglz](http://www.togglz.org), a Java framework for adding feature toggles in your application. Feature toggles are interesting when you're doing continuous deployment and kind of want to have some measure of control of which features are actually enabled for end users. They also allow you to do A/B testing on a backend level, where certain features are enabled for some users based on their location or other determining factors.

Togglz is actually a very nice framework with a lot of integrations, one of which is Spring Boot, which makes me very happy off course. Unfortunately, adding Togglz to your code is quite intrusive, as the examples give you instructions like this:

{% highlight java %}
if( MyFeatures.FEATURE_ONE.isActive() ) {
  // new stuff here
}
{% endhighlight %}

I don't know about you, but adding `if` statements all over my code doesn't make me a happy camper. This would also imply that I have to break a core principle when using Clean Architecture: keeping frameworks out of the application layer. Adding an `if` structure to my application layer implies having a direct dependency on Togglz, which is something I really want to avoid.

However, this looks like a prime example when you should use aspects. I also want to keep the dependency to Togglz out of my application layer, so I'll have to add some extra indirection.

So we'll start off with a simple enum that lists the features we can toggle.

{% highlight java %}
public enum MyFeature {
    MAKE_COFFEE,
    MAKE_TEA
}
{% endhighlight %}

Then we'll create a simple annotation that indicates which use cases in my application layer can be toggled.

{% highlight java %}
@Retention(RetentionPolicy.RUNTIME)
public @interface FeatureToggle {
	MyFeature value();
}
{% endhighlight %}

No dependencies to external frameworks, so we're good. Now I can use this in one of my use cases in my application layer.

{% highlight java %}
@FeatureToggle(MAKE_TEA)
public class MakeTeaImpl implements MakeTea {
    ...
}
{% endhighlight %}

So far, this code does nothing. The real magic is when you add an aspect that does all the heavy lifting. This code resides in your infrastructure layer, so dependencies here are not a problem (they are in fact the only layers that should have dependencies that bind you to a certain solution to a problem).

{% highlight java %}
@Aspect
public class FeatureToggleAspect {
	@Around("@within(featureToggle)")
	public Object execute(final ProceedingJoinPoint pjp, FeatureToggle featureToggle) throws Throwable {
		if (FeatureContext.getFeatureManager().isActive(new EnumFeatureWrapper(featureToggle.value()))) {
			return pjp.proceed();
		} else {
			throw new FeatureNotEnabledException("This feature was not enabled for this instance", featureToggle.value());
		}
	}
}
{% endhighlight %}

`FeatureNotEnabledException` is just a simple runtime exception, in case you were wondering.

Because I didn't use any Togglz dependencies in my application layer, I need to create a wrapper that will wrap my enum values into a `Feature` instance. This is what you see here being used as `EnumFeatureWrapper`.

{% highlight java %}
public class EnumFeatureWrapper implements Feature {
	private Enum<?> enumValue;

	public EnumFeatureWrapper(Enum<?> enumValue) {
		this.enumValue = enumValue;
	}

	@Override
	public String name() {
		return enumValue.name();
	}
}
{% endhighlight %}

If you now start your application with the `togglz-spring-boot-starter` dependency, in your Spring Boot application, you can now enjoy clean Togglz integration, but you'll notice a problem.

Because of the same reason that I needed to write the `EnumFeatureWrapper`, you'll have to write some extra code to have Togglz find your features. Togglz locates features by specifying a `FeatureProvider` implementation. These list the features you have can use and also provides some meta data on your features, like labels and such. The standard `EnumFeatureProvider` expects enums with Togglz annotations, which we don't have. So we'll need to write our own. Also the configuration system that tells you how features should be configured is a bit odd (has some problems when you're using YAML configuration)and is lacking some stuff (like labels and grouping), so we'll handle this as well.

We'll start off with writing our `FeatureProvider`.

{% highlight java %}
@Named
public class MyFeatureProvider implements FeatureProvider {

	private final Environment environment;

	public MyFeatureProvider(Environment environment) {
		this.environment = environment;
	}

	@Override
	public Set<Feature> getFeatures() {
		return Arrays.stream(MyFeature.values())
				.map(EnumFeatureWrapper::new)
				.collect(Collectors.toSet());
	}

	@Override
	public FeatureMetaData getMetaData(Feature feature) {
		return new EnvironmentFeatureMetaData(feature, environment);
	}
}
{% endhighlight %}

In addition, we need an `EnvironmentFeatureMetaData` class that does some configuration mojo that's a bit more in line what we would expect and that is compatible with YAML configuration.

{% highlight java %}
public class EnvironmentFeatureMetaData implements FeatureMetaData {
	private Feature feature;
	private Environment environment;

	public EnvironmentFeatureMetaData(Feature feature, Environment environment) {
		this.feature = feature;
		this.environment = environment;
	}

	@Override
	public String getLabel() {
		return environment.getProperty("togglz.features." + feature.name() + ".label", feature.name());
	}

	@Override
	public FeatureState getDefaultFeatureState() {
		boolean defaultEnabledState = environment.getProperty("togglz.features." + feature.name() + ".enabled", Boolean.class, false);
		return new FeatureState(feature, defaultEnabledState);
	}

	@Override
	public Set<FeatureGroup> getGroups() {
		String group = environment.getProperty("togglz.features." + feature.name() + ".group");
		final HashSet<FeatureGroup> featureGroups = new HashSet<>();
		if(group != null) {
			featureGroups.add(new SimpleFeatureGroup(group));
		}
		return featureGroups;
	}

	@Override
	public Map<String, String> getAttributes() {
		return new HashMap<>();
	}
}
{% endhighlight %}

Now we're in business. If your classpath scanning is set up correctly, startup should go smooth, but you'll notice that all your annotated use cases throw a `FeatureNotEnabledException` when invoked. That's because every feature is disabled by default (by the `EnvironmentFeatureMetaData`) unless configured as enabled.

So you'll have to add some configuration to your `application.yaml`.

{% highlight yaml %}
togglz:
  features:
    MAKE_COFFEE:
      enabled: true
      label: Making a cup of coffee
      group: Brewing
    MAKE_TEA:
      enabled: true
      label: Making a cup of tea
      group: Brewing
{% endhighlight %}

If you now start up your application, every `@FeatureToggle`-annotated class will work as expected. 

The `togglz-spring-boot-starter` dependency also registers a REST endpoint by default (`/togglz`) that allows you to see the state of your feature toggles. If you add the `togglz-console` dependency as well, you can go to `/togglz-console` in your browser an enable/disable features at runtime, as well as alter the activation strategies. If you want to know more about those, I suggest looking at the documentation.

As you can see, adding support for frameworks like Togglz requires a wee bit more effort when you want to adhere to Clean Architecture principles, but in the end it's worth it, as your code will be a lot cleaner. And should you choose to use another framework for feature toggling, you won't have to touch your application code, just the infrastructure layer.