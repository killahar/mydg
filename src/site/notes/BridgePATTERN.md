---
{"dg-publish":true,"permalink":"/bridge-pattern/"}
---



```Java
public class BarChartApp {  
  
    public static void main(String[] args) {  
        List<Pair> pairs = List.of(  
                new Pair("Mon", 16),  
                new Pair("Tue", 17),  
                new Pair("Wed", 19),  
                new Pair("Thu", 22),  
                new Pair("Fri", 24),  
                new Pair("Sat", 21),  
                new Pair("Sun", 19)  
        );  
  
        BarRenderer textRenderer = new TextBarRenderer();  
        BarCharter textBarCharter = new BarCharter(textRenderer);  
        textBarCharter.render("Forecast 6/8", pairs);  
  
        BarRenderer imageRenderer = new ImageBarRenderer();  
        BarCharter imageBarCharter = new BarCharter(imageRenderer);  
        imageBarCharter.render("Forecast 6/8", pairs);  
    }  
}  
  
abstract class BarRenderer {  
    public abstract void initialize(int bars, int maximum);  
    public abstract void drawCaption(String caption);  
    public abstract void drawBar(String name, int value);  
    public abstract void finalizeRender();  
}  
  
class Pair {  
    String name;  
    int value;  
  
    public Pair(String name, int value) {  
        this.name = name;  
        this.value = value;  
    }  
}  
  
class BarCharter {  
    private final BarRenderer renderer;  
  
    public BarCharter(BarRenderer renderer) {  
        this.renderer = renderer;  
    }  
  
    public void render(String caption, List<Pair> pairs) {  
        int maximum = pairs.stream().mapToInt(pair -> pair.value).max().orElse(1);  
        renderer.initialize(pairs.size(), maximum);  
        renderer.drawCaption(caption);  
        for (Pair pair : pairs) {  
            renderer.drawBar(pair.name, pair.value);  
        }  
        renderer.finalizeRender();  
    }  
}  
  
class TextBarRenderer extends BarRenderer {  
    private double scale;  
  
    @Override  
    public void initialize(int bars, int maximum) {  
        if (bars <= 0 || maximum <= 0) throw new IllegalArgumentException("Invalid bars or maximum value");  
        this.scale = 40.0 / maximum;  
    }  
  
    @Override  
    public void drawCaption(String caption) {  
        System.out.printf("%s%n%s%n", caption, "=".repeat(caption.length()));  
    }  
  
    @Override  
    public void drawBar(String name, int value) {  
        int scaledValue = (int) (value * scale);  
        System.out.printf("%s %s%n", "*".repeat(scaledValue), name);  
    }  
  
    @Override  
    public void finalizeRender() {  
        // Не требуется завершение для текстового рендерера  
    }  
}  
  
class ImageBarRenderer extends BarRenderer {  
    private final int stepHeight = 10;  
    private final int barWidth = 30;  
    private final int barGap = 5;  
    private BufferedImage image;  
    private Graphics2D g2d;  
    private int index = 0;  
    private final Color[] colors = {Color.RED, Color.GREEN, Color.BLUE, Color.YELLOW, Color.MAGENTA, Color.CYAN};  
    private String filename;  
  
    @Override  
    public void initialize(int bars, int maximum) {  
        int width = bars * (barWidth + barGap);  
        int height = maximum * stepHeight;  
        image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);  
        g2d = image.createGraphics();  
        g2d.setColor(Color.WHITE);  
        g2d.fillRect(0, 0, width, height);  
    }  
  
    @Override  
    public void drawCaption(String caption) {  
        filename = System.getProperty("java.io.tmpdir") + File.separator + caption.replaceAll("\\W+", "_") + ".png";  
    }  
  
    @Override  
    public void drawBar(String name, int value) {  
        g2d.setColor(colors[index % colors.length]);  
        int x = index * (barWidth + barGap);  
        int y = image.getHeight() - (value * stepHeight);  
        g2d.fillRect(x, y, barWidth, value * stepHeight);  
        g2d.setColor(Color.BLACK);  
        g2d.drawString(name, x + barWidth / 4, image.getHeight() - 5);  
        index++;  
    }  
  
    @Override  
    public void finalizeRender() {  
        try {  
            g2d.dispose();  
            ImageIO.write(image, "png", new File(filename));  
            System.out.println("Saved image to " + filename);  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
    }  
}
```
