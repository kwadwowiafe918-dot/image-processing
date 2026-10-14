# Arrays in Image Processing

## 1. The idea

A digital image is a grid of pixels. Each pixel is one number, so the whole image is stored as an **array**.

For a grayscale image, each value is a brightness from 0 (black) to 255 (white):

```
        col 0  col 1  col 2
row 0 [  10,    50,    90 ]
row 1 [  20,    60,   100 ]
row 2 [  30,    70,   110 ]
```

`image[1][2]` is row 1, column 2, which is `100`.

## 2. How the array is stored in memory

A 2D image is stored as one flat, contiguous block of memory. Pixel `(row, col)` is found with simple arithmetic:

```
index = row * width + col
```

Example: in an image 3 pixels wide, pixel (1, 2) is at index `1 * 3 + 2 = 5`.

Because the position is calculated rather than searched for, access takes constant time, O(1).

## 3. Operations on the array (C++)

### Brighten: visit every pixel
```cpp
for (auto& p : pixels)
    p = std::clamp(p + amount, 0, 255);
```

### Flip: reverse each row
```cpp
for (int r = 0; r < height; r++)
    for (int c = 0; c < width; c++)
        out.at(r, width - 1 - c) = at(r, c);
```

### Rotate 90 degrees: remap indices
Pixel `(r, c)` moves to `(c, height - 1 - r)`.
```cpp
out.at(c, height - 1 - r) = at(r, c);
```

### Blur: read neighboring pixels
Each output pixel is the average of the 3×3 block around it.
```cpp
for (int dr = -1; dr <= 1; dr++)
    for (int dc = -1; dc <= 1; dc++)
        total += at(r + dr, c + dc);   // neighbors found by index offset
```

### Histogram: an array used as a counter
A 256-element array counts how many pixels have each brightness. The pixel value is the index.
```cpp
std::array<int, 256> counts{};
for (auto p : pixels)
    counts[p]++;
```

## 4. Why arrays are necessary here

| Reason | Explanation |
|---|---|
| **Instant access** | Any pixel is reached by its index, with no searching |
| **Fast neighbor access** | Blur and edge detection read nearby pixels, which sit next to each other in memory |
| **Low memory use** | No pointers or links between pixels, so a million pixels cost about a million bytes |
| **Hardware friendly** | CPUs and GPUs process contiguous blocks very quickly |

A linked list would have to walk through thousands of nodes to reach a single pixel, which would make filters impractically slow.

## 5. Complexity

| Operation | Time | Space |
|---|---|---|
| Brighten, invert, flip, rotate | O(width × height) | O(width × height) |
| Box blur (3×3) | O(width × height × 9) | O(width × height) |
| Histogram | O(width × height) | O(256) |
| Access one pixel | O(1) | none extra |

Every operation must touch each pixel at least once, so O(width × height) is the best possible.

## 6. Limits of arrays

- The size is fixed once the array is created.
- Inserting or deleting in the middle is slow because other items must shift.

For images, neither matters: the size stays the same and the pixels are only read and rewritten in place.

## 7. Conclusion

Image processing is a natural fit for arrays because an image is already a grid of numbers. Arrays give instant access by position, fast reading of neighboring pixels, and very low memory overhead, which is why photo editors, medical imaging, and computer vision all rely on them.
