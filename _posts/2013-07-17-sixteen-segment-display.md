---
layout: post
category : "java"
title: "Repost: Creating a 16-segment character display component in Swing"
tags: [swing]
---

**Bump**: While I'm migrating my old blog content, I found this one from 3 years ago. It still looks funky, so I decided to give it a bump. Enjoy.

Some time ago, Javalobby posted a nice article on how someone created a event talk board that resembled [a departure board](http://www.jug-muenster.de/swing-event-departure-board-518/) at the airport or trainstation.

I needed to brush up my Java2D skills, so I started on something easy that was inspired by that article: a 16-segment character display (as described [here](http://en.wikipedia.org/wiki/Fourteen-segment_display)). It should only use Java2D (no images) and capable of showing A-Z and 0-9.<!--more-->

After an evening of drawing on paper grid, this is the end result:

![16 segment display](/images/sixteensegment.png)

The code is divided in 3 parts: the single character component, a character group component capable of displaying a String and the alfabet definition.

First I’ll show the single character component:

{% highlight java %}
public class SixteenSegmentDisplay extends JComponent {
    private java.util.List<Integer> lightedSegments;
    public static final int WIDTH = 29;
    public static final int HEIGHT = 46;

    private final double INACTIVE_COLOR_DARKEN_FRACTION = 0.4;

    private static final Image BACKGROUND_IMAGE = createBackgroundImage();

    // Highlight Colors
    private static final java.awt.Color HIGHLIGHT_COLOR_TOP = new java.awt.Color(0x000000);
    private static final java.awt.Color HIGHLIGHT_COLOR_BOTTOM = new java.awt.Color(0x625C52);

    private Color activeSegmentColor = new Color(0xFF0000);
    private Color inactiveSegmentColor = darken(new Color(0xFF0000), INACTIVE_COLOR_DARKEN_FRACTION);

    private Color backgroundColor = Color.black;

    public SixteenSegmentDisplay() {
        lightedSegments = new ArrayList<Integer>();
    }

    @Override
    public java.awt.Dimension getSize() {
        return new Dimension(WIDTH, HEIGHT);
    }

    @Override
    public java.awt.Dimension getPreferredSize() {
        return new Dimension(WIDTH, HEIGHT);
    }

    @Override
    public java.awt.Dimension getSize(java.awt.Dimension rv) {
        return new Dimension(WIDTH, HEIGHT);
    }

    public void setSegmentColor(Color segmentColor) {
        activeSegmentColor = segmentColor;
        inactiveSegmentColor = darken(segmentColor, INACTIVE_COLOR_DARKEN_FRACTION);
    }

    public void setBackgroundColor(Color backgroundColor) {
        this.backgroundColor = backgroundColor;
    }

    public void setCharacter(char character) {
        setLightedSegments(SixteenSegmentAlfabet.MAP.get(character));
    }

    private void setLightedSegments(SixteenSegmentAlfabet character) {
        if (character != null && character.getSegments() != null)
            this.lightedSegments = Arrays.asList(character.getSegments());
        else
            this.lightedSegments = new ArrayList<Integer>(0);
        this.repaint();
    }

    private Color darken(Color c, double fragment) {
        Double newRed = fragment * c.getRed();
        Double newBlue = fragment * c.getBlue();
        Double newGreen = fragment * c.getGreen();
        return new Color(newRed.intValue(), newGreen.intValue(), newBlue.intValue());
    }

    @Override
    public void paint(Graphics g) {
        Graphics2D graphics = (Graphics2D) g.create();
        graphics.setRenderingHint(java.awt.RenderingHints.KEY_ANTIALIASING, java.awt.RenderingHints.VALUE_ANTIALIAS_ON);
        graphics.setRenderingHint(java.awt.RenderingHints.KEY_ALPHA_INTERPOLATION, java.awt.RenderingHints.VALUE_ALPHA_INTERPOLATION_QUALITY);
        graphics.setRenderingHint(java.awt.RenderingHints.KEY_COLOR_RENDERING, java.awt.RenderingHints.VALUE_COLOR_RENDER_QUALITY);
        paintBackground(graphics);
        // left top to top middle
        paintSegment(graphics, lightedSegments.contains(1), getPoints("2,2;3,1;14,1;15,2;14,3;3,3;2,2"));
        // top middle to right top
        paintSegment(graphics, lightedSegments.contains(2), getPoints("15,2;16,1;27,1;28,2;27,3;16,3;15,2"));
        // left top to left middle
        paintSegment(graphics, lightedSegments.contains(3), getPoints("2,2;3,3;3,22;2,23;1,22;1,3;2,2"));
        // top left to middle middle
        paintSegment(graphics, lightedSegments.contains(4), getPoints("4,4;5,4;13,19;13,21;12,21;4,6;4,4"));
        // top middle to middle middle
        paintSegment(graphics, lightedSegments.contains(5), getPoints("15,2;14,3;14,22;15,23;16,22;16,3;15,2"));
        // top right to middle middle
        paintSegment(graphics, lightedSegments.contains(6), getPoints("25,4;26,4;26,6;18,21;17,21;17,19;25,4"));
        // top middle to right middle
        paintSegment(graphics, lightedSegments.contains(7), getPoints("28,2;29,3;29,22;28,23;27,22;27,3;28,2"));
        // left middle to middle middle
        paintSegment(graphics, lightedSegments.contains(8), getPoints("2,23;3,22;14,22;15,23;14,24;3,24;2,23"));
        // middle middle to right middle
        paintSegment(graphics, lightedSegments.contains(9), getPoints("15,23;16,22;27,22;28,23;27,24;16,24;15,23"));
        // left middle to bottom left
        paintSegment(graphics, lightedSegments.contains(10), getPoints("2,23;3,24;3,43;2,44;1,43;1,24;2,23"));
        // left bottom to middle middle
        paintSegment(graphics, lightedSegments.contains(11), getPoints("13,27;13,25;12,25;4,40;4,42;5,42;13,27"));
        // bottom middle to middle middle
        paintSegment(graphics, lightedSegments.contains(12), getPoints("15,23;16,24;16,43;15,44;14,43;14,24;15,23"));
        // bottom right to middle middle
        paintSegment(graphics, lightedSegments.contains(13), getPoints("17,25;18,25;26,40;26,42;25,42;17,27;17,25"));
        // right middle to bottom right
        paintSegment(graphics, lightedSegments.contains(14), getPoints("28,23;29,24;29,43;28,44;27,43;27,24;28,23"));
        // bottpm left to bottom middle
        paintSegment(graphics, lightedSegments.contains(15), getPoints("2,44;3,43;14,43;15,4;14,45;3,45;2,44"));
        // bottom middle to bottom right
        paintSegment(graphics, lightedSegments.contains(16), getPoints("15,244;16,43;27,43;28,44;27,45;16,45;15,44"));
    }

    private void paintBackground(Graphics2D g) {
        g.drawImage(BACKGROUND_IMAGE, 0, 0, this);
    }

    private static final BufferedImage createBackgroundImage() {
        GraphicsConfiguration gfxConf = GraphicsEnvironment.getLocalGraphicsEnvironment().getDefaultScreenDevice().getDefaultConfiguration();
        final BufferedImage IMAGE = gfxConf.createCompatibleImage(WIDTH, HEIGHT, Transparency.OPAQUE);
        Graphics2D g2 = IMAGE.createGraphics();
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        g2.setRenderingHint(RenderingHints.KEY_ALPHA_INTERPOLATION, RenderingHints.VALUE_ALPHA_INTERPOLATION_QUALITY);
        g2.setRenderingHint(RenderingHints.KEY_COLOR_RENDERING, RenderingHints.VALUE_COLOR_RENDER_QUALITY);
        g2.setRenderingHint(RenderingHints.KEY_STROKE_CONTROL, RenderingHints.VALUE_STROKE_PURE);
        final java.awt.geom.Point2D START_INNER_BACKGROUND = new java.awt.geom.Point2D.Float(0, 0);
        final java.awt.geom.Point2D STOP_INNER_BACKGROUND = new java.awt.geom.Point2D.Float(0, HEIGHT);
        final float[] FRACTIONS_INNER_BACKGROUND =
                {
                        0.0f,
                        1.0f
                };
        final Color[] COLORS_INNER_BACKGROUND =
                {
                        new Color(0x3D3A31),
                        new Color(0x232520)
                };
        final LinearGradientPaint GRADIENT_INNER_BACKGROUND = new LinearGradientPaint(START_INNER_BACKGROUND, STOP_INNER_BACKGROUND, FRACTIONS_INNER_BACKGROUND, COLORS_INNER_BACKGROUND);
        g2.setPaint(GRADIENT_INNER_BACKGROUND);
        g2.fill(new java.awt.geom.Rectangle2D.Float(0, 0, WIDTH, HEIGHT));
        // Highlight
        g2.setColor(HIGHLIGHT_COLOR_TOP);
        g2.drawLine(0, 0, WIDTH, 0);
        g2.setColor(HIGHLIGHT_COLOR_BOTTOM);
        g2.drawLine(0, HEIGHT, WIDTH, HEIGHT);
        g2.dispose();
        return IMAGE;
    }

    private void paintSegment(Graphics2D g, boolean active, Point... points) {
        if (points != null) {
            g.setColor(active ? activeSegmentColor : inactiveSegmentColor);
            Polygon polygon = new Polygon();
            for (Point point : points) {
                polygon.addPoint(point.x, point.y);
            }
            g.fillPolygon(polygon);
        }
    }

    private static Point[] getPoints(String coded) {
        if(coded.length() == 0)
        {
            return null;
        }
        java.util.List<Point> pointList = new ArrayList<Point>();
        String[] points = coded.split(";");
        for (String point : points) {
            String[] pointCoordinates = point.split(",");
            pointList.add(new Point(Integer.parseInt(pointCoordinates[0]), Integer.parseInt(pointCoordinates[1])));
        }
        return pointList.toArray(new Point[pointList.size()]);
    }
}
{% endhighlight %}

To use this, you need an alfabet definition that defines which segments should light up for a given character:

{% highlight java %}
public enum SixteenSegmentAlfabet {
    A(new char[]{'a', 'A'}, 1, 2, 3, 7, 8, 9, 10, 14),
    B(new char[]{'b', 'B'}, 1, 2, 5, 7, 9, 12, 14, 15, 16),
    C(new char[]{'c', 'C'}, 1, 2, 3, 10, 15, 16),
    D(new char[]{'d', 'D'}, 1, 2, 5, 7, 12, 14, 15, 16),
    E(new char[]{'e', 'E'}, 1, 2, 3, 8, 9, 10, 15, 16),
    F(new char[]{'f', 'F'}, 1, 2, 3, 8, 10),
    G(new char[]{'g', 'G'}, 1, 2, 3, 9, 10, 14, 15, 16),
    H(new char[]{'h', 'H'}, 3, 7, 8, 9, 10, 14),
    I(new char[]{'i', 'I'}, 1, 2, 5, 12, 15, 16),
    J(new char[]{'j', 'J'}, 7, 10, 14, 15, 16),
    K(new char[]{'k', 'K'}, 3, 6, 8, 10, 13),
    L(new char[]{'l', 'L'}, 3, 10, 15, 16),
    M(new char[]{'m', 'M'}, 3, 4, 6, 7, 10, 14),
    N(new char[]{'n', 'N'}, 3, 4, 7, 10, 13, 14),
    O(new char[]{'o', 'O'}, 1, 2, 3, 7, 10, 14, 15, 16),
    P(new char[]{'p', 'P'}, 1, 2, 3, 6, 8, 10),
    Q(new char[]{'q', 'Q'}, 1, 2, 3, 7, 10, 13, 14, 15, 16),
    R(new char[]{'r', 'R'}, 1, 2, 3, 7, 8, 9, 10, 13),
    S(new char[]{'s', 'S'}, 1, 2, 3, 8, 9, 14, 15, 16),
    T(new char[]{'t', 'T'}, 1, 2, 5, 12),
    U(new char[]{'u', 'U'}, 3, 7, 10, 14, 15, 16),
    V(new char[]{'v', 'V'}, 3, 6, 10, 11),
    W(new char[]{'w', 'W'}, 3, 7, 10, 11, 13, 14),
    X(new char[]{'x', 'X'}, 4, 6, 11, 13),
    Y(new char[]{'y', 'Y'}, 4, 6, 12),
    Z(new char[]{'z', 'Z'}, 1, 2, 6, 11, 15, 16),
    BLANK(new char[]{' '}, null),
    ZERO(new char[]{'0'}, 1, 2, 3, 4, 7, 10, 13, 14, 15, 16),
    ONE(new char[]{'1'}, 7, 14),
    TWO(new char[]{'2'}, 1, 2, 7, 8, 9, 10, 15, 16),
    THREE(new char[]{'3'}, 1, 2, 7, 9, 14, 15, 16),
    FOUR(new char[]{'4'}, 3, 7, 8, 9, 14),
    FIVE(new char[]{'5'}, 1, 2, 3, 8, 13, 15, 16),
    SIX(new char[]{'6'}, 1, 2, 3, 8, 9, 10, 14, 15, 16),
    SEVEN(new char[]{'7'}, 1, 2, 7, 14),
    EIGHT(new char[]{'8'}, 1, 2, 3, 7, 8, 9, 10, 14, 15, 16),
    NINE(new char[]{'9'}, 1, 2, 3, 7, 8, 9, 14, 15, 16);
    private final Integer[] segments;
    private final char[] characters;

    public static final Map<Character, SixteenSegmentAlfabet> MAP;

    static {
        MAP = new HashMap<Character, SixteenSegmentAlfabet>(36);
        for (SixteenSegmentAlfabet alfabet : SixteenSegmentAlfabet.values()) {
            for (char character : alfabet.characters) {
                MAP.put(character, alfabet);
            }
        }
    }

    SixteenSegmentAlfabet(char[] characters, Integer... segments) {
        this.characters = characters;
        this.segments = segments;
    }

    public Integer[] getSegments() {
        return segments;
    }
}
{% endhighlight %}

To finish, the component capable of showing a String merely groups together a couple of the character components:

{% highlight java %}
public class MultiCharacterDisplay extends JComponent {
    private static final int GAP = 5;
    private int segmentDisplayCount;
    private List<SixteenSegmentDisplay> segmentDisplays;

    public MultiCharacterDisplay(int segmentDisplayCount) {
        this(segmentDisplayCount, Color.red);
    }

    public MultiCharacterDisplay(int segmentDisplayCount, Color segmentColor) {
        this.segmentDisplayCount = segmentDisplayCount;
        segmentDisplays = new ArrayList<SixteenSegmentDisplay>(segmentDisplayCount);
        setLayout(new FlowLayout(FlowLayout.LEFT, GAP, 0));
        for (int i = 0; i < segmentDisplayCount; i++) {
            SixteenSegmentDisplay display = new SixteenSegmentDisplay();
            display.setSegmentColor(segmentColor);
            segmentDisplays.add(display);
            add(display);
        }
    }

    public void setText(String text) {
        if (text.length() > segmentDisplayCount) {
            text = text.substring(0, segmentDisplayCount);
        }
        int i = 0;
        for (char c : text.toCharArray()) {
            segmentDisplays.get(i++).setCharacter(c);
        }
    }
	
	public static void main(String[] args) {
        String text = args[0];

        MultiCharacterDisplay display = new MultiCharacterDisplay(text.length(), Color.GREEN);
        display.setText(text);

        JFrame frame = new JFrame();
        frame.setBackground(Color.black);
        frame.getContentPane().setBackground(Color.black);
        frame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
        frame.getContentPane().add(display);
        frame.pack();

        frame.setVisible(true);
    }
}
{% endhighlight %}

All in all, it was a nice exercise :) . If I find some more time, I’ll try and build some sort of marquee with it.
