# Word-Search-Generator-in-Java

```
package WordSearchTut;

import java.util.List;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Random;
import java.util.Scanner;

public class WordSearchGenerator {
	
	// Grid of the word search game
	public static class Grid {
		int numAttempts;
		char[][] cells = new char[nRows][nCols];
		List<String> solutions = new ArrayList<>();
	}
	
	// 8 directions to generate words in the grid
	public static final int[][] DIRS = {
		{1, 0}, {0, 1}, {1, 1}, {1, -1}, {-1, 0}, {0, -1}, {-1, -1}, {-1, 1}
	};
	
	// nb rows and cols for the grid
	public static final int nRows = 10, nCols = 10;
	public static final int gridSize = nRows * nCols;
	
	// min number of words to place on the grid to generate
	public static final int minWords = 25;
	
	public static final Random RANDOM = new Random();
	
	public static void main(String[] args) {
		printResult(createWordSearch(readWords("unixdict.txt")));
	}
	

	public static List<String> readWords(String filename) {
		int maxLength = Math.max(nRows, nCols);
		List<String> words = new ArrayList<>();
		
		try (Scanner sc = new Scanner(new FileReader(filename))) {
			while(sc.hasNext()) {
				String s = sc.next().trim().toLowerCase();
				
				// only pick the words with length between 3 and maxLength and with a-z inside
				if (s.matches("^[a-z]{3," + maxLength + "}$")) {
					words.add(s.toUpperCase());
				}
			}
		} catch (FileNotFoundException e) {
			// Manage error
		}
		
		return words;
	}
	
	public static Grid createWordSearch(List<String> words) {
		Grid grid = null;
		int numAttempts = 0;
		
		// 100 attempts to generate a grid
		while(++numAttempts < 100) {
			// Shuffle the words
			Collections.shuffle(words);
			
			grid = new Grid();
			int messageLength = placeMessage(grid, "ian mobile app");
			int target = gridSize - messageLength;
			int cellsFilled = 0;
			
			for (String word : words) {
				cellsFilled += tryPlaceWord(grid, word);
				if (cellsFilled == target) {
					if (grid.solutions.size() >= minWords) {
						grid.numAttempts = numAttempts;
						return grid;
					} else {
						// Fulfilled the grid but we have not enough words, we start over
						break;
					}
				}
			}
		}
		
		return grid;
	}
	
	public static int placeMessage(Grid grid, String msg) {
		msg = msg.toUpperCase().replaceAll("[^A-Z]", "");
		int messageLength = msg.length();
		
		if (messageLength > 0 && messageLength < gridSize) {
			int gapSize = gridSize / messageLength;
			
			for (int i = 0; i < messageLength; i++) {
				int pos = i * gapSize + RANDOM.nextInt(gapSize);
				grid.cells[pos/nCols][pos%nCols] = msg.charAt(i);
			}
			
			return messageLength;
		}
		return 0;
	}
	
	public static int tryPlaceWord(Grid grid, String word) {
		int randDir = RANDOM.nextInt(DIRS.length);
		int randPos = RANDOM.nextInt(gridSize);
		
		for (int dir = 0; dir < DIRS.length; dir++) {
			dir = (dir + randDir) % DIRS.length;
			
			for (int pos = 0; pos < gridSize; pos++) {
				pos = (pos + randPos) % gridSize;
				
				int lettersPlaced = tryLocation(grid, word, dir, pos);
				
				if (lettersPlaced > 0) {
					return lettersPlaced;
				}
			}
		}
		
		return 0;
	}
	
	public static int tryLocation(Grid grid, String word, int dir, int pos) {
		int r = pos / nCols;
		int c = pos % nCols;
		int length = word.length();
		
		// Check bounds
		if (   (DIRS[dir][0] == 1 && (length + c) > nCols)
			|| (DIRS[dir][0] == -1 && (length - 1) > c)
			|| (DIRS[dir][1] == 1 && (length + r) > nRows)
			|| (DIRS[dir][1] == -1 && (length - 1) > r)) {
			return 0;
		}
		
		int i, rr, cc, overlaps = 0;
		
		// Check Cells
		for (i = 0, rr = r, cc = c; i < length; i++) {
			if (grid.cells[rr][cc] != 0 && grid.cells[rr][cc] != word.charAt(i)) {
				return 0;
			}
			
			cc += DIRS[dir][0];
			rr += DIRS[dir][1];
		}
		
		// Place word
		for (i = 0, rr = r, cc = c; i < length; i++) {
			if (grid.cells[rr][cc] == word.charAt(i)) {
				overlaps++;
			} else {
				grid.cells[rr][cc] = word.charAt(i);
			}
			
			if (i < length - 1) {
				cc += DIRS[dir][0];
				rr += DIRS[dir][1];
			}
		}
		
		int lettersPlaced = length - overlaps;
		
		if (lettersPlaced > 0) {
			grid.solutions.add(String.format("%-10s (%d,%d)(%d,%d)", word, c, r, cc, rr));
		}
		
		return lettersPlaced;
	}
		
	// Print result method
	public static void printResult(Grid grid) {
		if (grid == null || grid.numAttempts == 0) {
			System.out.println("No grid to display");
			return;
		}
		
		int size = grid.solutions.size();
		
		System.out.println("Number of Attempts: " + grid.numAttempts);
		System.out.println("Number of Words: " + size);
		System.out.print("\n    ");
		
		for (int c = 0; c < nCols; c++) {
			System.out.print(c + "  ");
		}
		
		System.out.println();
		
		for (int r = 0; r < nRows; r++) {
			System.out.printf("%n%d  ", r);
			
			for (int c = 0; c < nCols; c++) {
				System.out.printf(" %c ", grid.cells[r][c]);
			}
		}
		
		System.out.println("\n");
		
		for (int i = 0; i < size - 1; i += 2) {
			System.out.printf("%s  %s%n", grid.solutions.get(i), grid.solutions.get(i + 1));
		}
		
		if (size % 2 == 1) {
			System.out.println(grid.solutions.get(size - 1));
		}
	}

}

```

![image](https://github.com/kaian0414/Word-Search-Generator-in-Java/blob/main/WordSearchGenerator_v2.png)
