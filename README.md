import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.Dimension;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.RenderingHints;
import java.awt.image.BufferedImage;
import javax.swing.JFrame;
import javax.swing.JPanel;

/**
 * EscapeBall run program that...
 * @author User | 04/07/2024
 */
public class EscapeBall extends JFrame
{
    private JPanel canvas;
    public static final double TARGET_FPS = 120.0;
    public static final double TARGET_TIME = 1000000000 / TARGET_FPS; // in nanoseconds
    private int Cwidth, Cheight;
    private double BallSpeed = 2;

    private Circle circle;
    private Ball ball;

    public EscapeBall(int width, int height)
    {
        this.Cheight = height;
        this.Cwidth = width;
        super.setTitle("Escape Ball");
        super.setResizable(false);
        super.setIgnoreRepaint(true);
        super.setSize(width, height);
        super.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        this.canvas = new JPanel();
        this.canvas.setPreferredSize(new Dimension(width, height));
        this.canvas.setIgnoreRepaint(true);
        this.canvas.setFocusable(true);

        super.add(this.canvas, BorderLayout.CENTER);
        super.pack();
        super.setLocationRelativeTo(null);
        super.setVisible(true);
    }

    public void start()
    {
        init();
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                runEngineLoop();
            }
        }).start();
    }

    private void runEngineLoop()
    {
        double delta = 0;
        int fps = 0, ups = 0;
        long lastTime = System.nanoTime(), timer = 0;

        while (true)
        {
            long currentTime = System.nanoTime();
            delta += (currentTime - lastTime) / TARGET_TIME;
            lastTime = currentTime;
            fps++;

            while (delta >= 1)
            {
                // update keyboard & mouse status
                canvas.requestFocus();
                update();
                renderAndDraw();
                delta--;
                ups++;
            }

            // one sec passed (update fps,ups,timer for debug)
            if (ups >= TARGET_FPS)
            {
                timer++;
                ups = 0;
                fps = 0;
            }
            // sleep some time to free CPU (let other threads CPU time)
            try
            {
                Thread.sleep(1);
            } catch (InterruptedException e)
            {
            }
        }
    }

    private void renderAndDraw()
    {
        BufferedImage bufferedImage = new BufferedImage(canvas.getWidth(), canvas.getHeight(), BufferedImage.TYPE_INT_ARGB);

        // create offscreen drawing surface using BufferedImage
        Graphics2D offg = (Graphics2D) bufferedImage.getGraphics();
        offg.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);

        // claer offscreen
        offg.setColor(Color.BLACK);
        offg.fillRect(0, 0, canvas.getWidth(), canvas.getHeight());

        // call sub-class draw on the offscreen drawing surface
        draw(offg);

        // draw offg on canvas
        canvas.getGraphics().drawImage(bufferedImage, 0, 0, null);
    }

    private void init()
    {
        circle = new Circle(400, 300, 190, Color.red);
        ball = new Ball(400, 300, 25, BallSpeed, BallSpeed, Color.YELLOW);
    }

    private void update()
    {
        ball.move(Cwidth, Cheight);
        checkColisionWCircle();
        gravityForce();
    }

    private void draw(Graphics2D offg)
    {
        circle.draw(offg);
        ball.draw(offg);
    }

    private void checkColisionWCircle()
    {
        double circleX = circle.getX();
        double circleY = circle.getY();
        double circleRadius = circle.getRadius();

        double ballX = ball.getX();
        double ballY = ball.getY();
        double ballRadius = ball.getRadius();

        double distance = Math.sqrt(Math.pow(ballX - circleX, 2) + Math.pow(ballY - circleY, 2));

        if (distance + ballRadius > circleRadius)
        {
            //for the Y
            if (ball.getY() + ball.getRadius() < 300)
            {
                ball.setDy(BallSpeed);
            } else
            {
                ball.setDy(-BallSpeed);
            }

            //for the X
            if (ball.getX() <= 400 + ball.getRadius() && ball.getX() >= 400 - ball.getRadius())
            {
                if (ball.getDx() <= 0)
                {
                    ball.setDx(-BallSpeed);
                } else
                {
                    ball.setDx(BallSpeed);
                }
            } else
            {
                if (ball.getDx() <= 0)
                {
                    ball.setDx(BallSpeed);
                } else
                {
                    ball.setDx(-BallSpeed);
                }
            }
        }
    }

    private void gravityForce()
    {
        //for the y
        ball.setDy(ball.getDy() + 0.02);
        //for the x
        if (ball.getDx() <= 0)
        {
            ball.setDx(ball.getDx() + 0.002);
        } else
        {
            ball.setDx(ball.getDx() - 0.002);
        }
    }

    public static void main(String[] args)
    {
        EscapeBall escapeball = new EscapeBall(800, 600);
        escapeball.start();
    }

}
