# Welcome-To

Copy and paste code below into java program, make sure to create package/class names in your program of choice so that it works correctly!

Press l then click to create starting spot for line, then l click to set the end position, be careful as lines cant be deleted in current version!
Press t to create text box you can use backspace! press enter when you are finished editing your text! (said text can also hold numbers)

The lines are used for both crossing everything out as well as creating fences and marking the board as complete
The deck auto shuffles etc so just play a normal round of Welcome to! enjoy!
all imports are needed, you can just also put "import java.util.*" and that should import all java resources.

Code:


package gameBoardRedered;

import java.awt.BasicStroke;
import java.awt.Color;
import java.awt.Font;
import java.awt.FontMetrics;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.Image;
import java.awt.Point;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.util.*;

import javax.imageio.ImageIO;
import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JOptionPane;
import javax.swing.JPanel;

public class GameBoardDisplay extends JPanel {

    private BufferedImage gameBoardImage;
    private List<Line> lines = new ArrayList<>();
    private List<TextLabel> textLabels = new ArrayList<>();
    private int startX = -1, startY = -1;
    private boolean drawingLine = false;

    private boolean typingText = false;
    private StringBuilder currentText = new StringBuilder();
    private int textX = -1, textY = -1;

    private List<Card> allCardImages = new ArrayList<>();
    private List<Card> deck = new ArrayList<>();
    private List<Card> discardPile = new ArrayList<>();
    private List<Card> currentCards = new ArrayList<>();
    private List<Card> previousCards = new ArrayList<>();

    private Map<String, BufferedImage> cardBacksMap = new HashMap<>();

    private JButton shuffleButton;
    private JButton nextButton;

    private final int CARD_WIDTH = 150;
    private final int CARD_HEIGHT = 225;
    private final int RIGHT_PADDING = 40;

    public GameBoardDisplay() {
        try {
            gameBoardImage = ImageIO.read(new File("C:/Users/kalebchristie/Downloads/WelcomeTo.png"));
        } catch (IOException e) {
            JOptionPane.showMessageDialog(this, "Error loading game board image: " + e.getMessage(), "Image Load Error", JOptionPane.ERROR_MESSAGE);
        }

        loadCardImages("C:/Users/kalebchristie/Downloads/Welcome To/Welcome To");
        shuffleDeck();

        setFocusable(true);
        setLayout(null);

        int boardWidth = (gameBoardImage != null) ? gameBoardImage.getWidth() : 800;
        int buttonX = boardWidth + RIGHT_PADDING;

        shuffleButton = new JButton("Shuffle Deck");
        shuffleButton.setBounds(buttonX, 20, 150, 30);
        add(shuffleButton);
        shuffleButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                reshuffleDeck();
                repaint();
            }
        });

        nextButton = new JButton("Next Cards");
        nextButton.setBounds(buttonX, 60, 150, 30);
        add(nextButton);
        nextButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                drawNextCards();
                repaint();
            }
        });

        addKeyListener(new KeyAdapter() {
            @Override
            public void keyPressed(KeyEvent e) {
                if (typingText) {
                    if (e.getKeyCode() == KeyEvent.VK_ENTER) {
                        if (currentText.length() > 0) {
                            textLabels.add(new TextLabel(currentText.toString(), textX, textY));
                        }
                        currentText.setLength(0);
                        typingText = false;
                        repaint();
                    } else if (e.getKeyCode() == KeyEvent.VK_BACK_SPACE && currentText.length() > 0) {
                        currentText.deleteCharAt(currentText.length() - 1);
                        repaint();
                    } else if (Character.isLetterOrDigit(e.getKeyChar()) || Character.isSpaceChar(e.getKeyChar())) {
                        currentText.append(e.getKeyChar());
                        repaint();
                    }
                } else {
                    if (e.getKeyChar() == 'l' || e.getKeyChar() == 'L') {
                        drawingLine = true;
                        requestFocusInWindow();
                    } else if (e.getKeyChar() == 't' || e.getKeyChar() == 'T') {
                        Point mousePos = getMousePosition();
                        if (mousePos != null) {
                            textX = mousePos.x;
                            textY = mousePos.y;
                            typingText = true;
                            currentText.setLength(0);
                            requestFocusInWindow();
                            repaint();
                        }
                    }
                }
            }
        });

        addMouseListener(new MouseAdapter() {
            @Override
            public void mouseClicked(MouseEvent e) {
                if (drawingLine) {
                    if (startX == -1) {
                        startX = e.getX();
                        startY = e.getY();
                    } else {
                        lines.add(new Line(startX, startY, e.getX(), e.getY()));
                        startX = -1;
                        startY = -1;
                        drawingLine = false;
                        repaint();
                    }
                }
            }
        });
    }

    private void loadCardImages(String folderPath) {
        File folder = new File(folderPath);
        if (!folder.exists() || !folder.isDirectory()) {
            JOptionPane.showMessageDialog(this, "Card image folder not found: " + folderPath, "Load Error", JOptionPane.ERROR_MESSAGE);
            return;
        }

        File[] files = folder.listFiles();
        if (files == null) return;

        for (File file : files) {
            String name = file.getName().toLowerCase();
            if (!name.endsWith(".png")) continue;

            try {
                BufferedImage img = ImageIO.read(file);
                if (img == null) continue;

                if (name.startsWith("b_")) {
                    String type = name.substring(2, name.indexOf(".png"));
                    cardBacksMap.put(type.toLowerCase(), img);
                } else {
                    allCardImages.add(new Card(img, name));
                }
            } catch (IOException e) {
                System.err.println("Failed to load image: " + name);
            }
        }
    }

    private void shuffleDeck() {
        deck.clear();
        discardPile.clear();
        previousCards.clear();
        deck.addAll(allCardImages);
        Collections.shuffle(deck);
        drawNextCards();
    }

    private void reshuffleDeck() {
        deck.addAll(discardPile);
        discardPile.clear();
        Collections.shuffle(deck);
        drawNextCards();
    }

    private void drawNextCards() {
        if (deck.size() < 3) {
            reshuffleDeck();
        }

        previousCards.clear();
        previousCards.addAll(currentCards);
        currentCards.clear();

        for (int i = 0; i < 3 && !deck.isEmpty(); i++) {
            Card card = deck.remove(0);
            currentCards.add(card);
            discardPile.add(card);
        }
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2d = (Graphics2D) g;

        if (gameBoardImage != null) {
            g2d.drawImage(gameBoardImage, 0, 0, this);
        }

        g2d.setColor(Color.BLACK);
        g2d.setStroke(new BasicStroke(3));
        for (Line line : lines) {
            g2d.drawLine(line.x1, line.y1, line.x2, line.y2);
        }

        if (drawingLine && startX != -1) {
            Point mouse = getMousePosition();
            if (mouse != null) {
                g2d.setColor(Color.GRAY);
                g2d.setStroke(new BasicStroke(1));
                g2d.drawLine(startX, startY, mouse.x, mouse.y);
            }
        }

        g2d.setColor(Color.BLACK);
        g2d.setFont(new Font("Arial", Font.BOLD, 48));
        for (TextLabel label : textLabels) {
            drawCenteredText(g2d, label.text, label.x, label.y);
        }

        if (typingText && currentText.length() > 0) {
            g2d.setColor(Color.BLUE);
            drawCenteredText(g2d, currentText.toString(), textX, textY);
        }

        int baseX = (gameBoardImage != null ? gameBoardImage.getWidth() : 800) + RIGHT_PADDING;
        int cardYStart = 120;

        for (int i = 0; i < currentCards.size(); i++) {
            int y = cardYStart + i * (CARD_HEIGHT + 30);
            Card front = currentCards.get(i);
            BufferedImage back = (i < previousCards.size()) ? getBackForCard(previousCards.get(i)) : null;

            if (front != null && front.image != null) {
                Image scaledFront = front.image.getScaledInstance(CARD_WIDTH, CARD_HEIGHT, Image.SCALE_SMOOTH);
                g2d.drawImage(scaledFront, baseX, y, this);
            }

            if (back != null) {
                Image scaledBack = back.getScaledInstance(CARD_WIDTH, CARD_HEIGHT, Image.SCALE_SMOOTH);
                g2d.drawImage(scaledBack, baseX + CARD_WIDTH + 20, y, this);
            }
        }
    }

    private BufferedImage getBackForCard(Card card) {
        String filename = card.filename.toLowerCase().replace(".png", "");
        String[] parts = filename.split("_");
        if (parts.length == 2) {
            String type = parts[1];
            BufferedImage back = cardBacksMap.get(type);
            if (back == null) {
                System.err.println("No back found for type \"" + type + "\"; using Fence fallback.");
                back = cardBacksMap.get("fence");
            }
            return back;
        }
        System.err.println("Could not parse card filename: " + filename + "; using Fence fallback.");
        return cardBacksMap.get("fence");
    }

    private void drawCenteredText(Graphics2D g2d, String text, int x, int y) {
        FontMetrics fm = g2d.getFontMetrics();
        int textWidth = fm.stringWidth(text);
        int textHeight = fm.getAscent();
        g2d.drawString(text, x - textWidth / 2, y + textHeight / 2);
    }

    private static class Line {
        int x1, y1, x2, y2;

        public Line(int x1, int y1, int x2, int y2) {
            this.x1 = x1; this.y1 = y1; this.x2 = x2; this.y2 = y2;
        }
    }

    private static class TextLabel {
        String text;
        int x, y;

        public TextLabel(String text, int x, int y) {
            this.text = text;
            this.x = x;
            this.y = y;
        }
    }

    private static class Card {
        BufferedImage image;
        String filename;

        public Card(BufferedImage image, String filename) {
            this.image = image;
            this.filename = filename;
        }
    }

    public static void main(String[] args) {
        JFrame frame = new JFrame("Game Board - Welcome To!");
        GameBoardDisplay gameBoardDisplay = new GameBoardDisplay();
        frame.add(gameBoardDisplay);

        int width = (gameBoardDisplay.gameBoardImage != null ? gameBoardDisplay.gameBoardImage.getWidth() : 800) + 500;
        int height = (gameBoardDisplay.gameBoardImage != null ? gameBoardDisplay.gameBoardImage.getHeight() : 600);
        frame.setSize(width, height);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setVisible(true);
    }
}

