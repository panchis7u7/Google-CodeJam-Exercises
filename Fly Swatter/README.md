# Problem

What are your chances of hitting a fly with a tennis racquet?

To start with, ignore the racquet's handle. Assume the racquet is a perfect ring, of outer radius R and thickness t (so the inner radius of the ring is R−t).

The ring is covered with horizontal and vertical strings. Each string is a cylinder of radius r. Each string is a chord of the ring (a straight line connecting two points of the circle). There is a gap of length g between neighbouring strings. The strings are symmetric with respect to the center of the racquet i.e. there is a pair of strings whose centers meet at the center of the ring.

The fly is a sphere of radius f. Assume that the racquet is moving in a straight line perpendicular to the plane of the ring. Assume also that the fly's center is inside the outer radius of the racquet and is equally likely to be anywhere within that radius. Any overlap between the fly and the racquet (the ring or a string) counts as a hit.

![Racket](https://codejam.googleapis.com/dashboard/get_file/AQj_6U0mWZ1I4-zFNJrEsKwit3HPdqMkHwEg2d9mTCEtOjcsJJfZ35TG9unV6w/test2.png)

### Input

One line containing an integer N, the number of test cases in the input file.

The next N lines will each contain the numbers f, R, t, r and g separated by exactly one space. Also the numbers will have exactly 6 digits after the decimal point.
Output

N lines, each of the form "Case #k: P", where k is the number of the test case and P is the probability of hitting the fly with a piece of the racquet.

Answers with a relative or absolute error of at most 10-6 will be considered correct.
Limits

Time limit: 60 seconds per test set.
Memory limit: 1GB.
f, R, t, r and g will be positive and smaller or equal to 10000.
t < R
f < R
r < R
Small dataset (Test set 1 - Visible)

1 ≤ N ≤ 30
The total number of strings will be at most 60 (so at most 30 in each direction).
Large dataset (Test set 2 - Hidden)

1 ≤ N ≤ 100
The total number of strings will be at most 2000 (so at most 1000 in each direction).
Sample

### Input
  	

5
0.250000 1.000000 0.100000 0.010000 0.500000
0.250000 1.000000 0.100000 0.010000 0.900000
0.000010 10000.000000 0.000010 0.000010 1000.000000
0.400000 10000.000000 0.000010 0.000010 700.000000
1.000000 100.000000 1.000000 1.000000 10.000000

  



### Output
 

Case #1: 1.000000
Case #2: 0.910015
Case #3: 0.000000
Case #4: 0.002371
Case #5: 0.573972

# Analysis

 The first step in solving this problem involves a standard trick. Instead of looking for the region that will hit the fly's circular body, we look for the region where the center of the fly must avoid. The problem is thus transformed into an equivalent one where we thicken the size of the racquet to t + f and the radius of the strings from r to r + f. And we consider the fly to be simply a single point, only focusing on its center.

Now the main problem is to find the area of the holes (the areas between the strings, or between the strings and the racquet), and divide it by the area of the disc corresponding to the original racquet. It is easy to see this gives the probability that the racquet does not hit the fly.

A simplifying observation is that we can just look at the first quadrant of the circle, because the racquet is symmetrical.

Now we can go through each square gap in the racquet and check a few cases. Each case consists in computing the area of intersection between the square and the circle representing the inside of the racquet. The case where the square is totally inside or totally outside the circle is easy. The cases left when one corner, two or three corners of the square are inside the circle and the other corners are outside. There are no other cases because we just look at the intersection of a quarter of the circle with small squares.

Each of these cases can be solved by splitting the intersection area into triangles and a circular segment. Here's some code that does this:

```c++
double circle_segment(double rad, double th) {
  return rad*rad*(th - sin(th))/2;
}

double rad = R-t-f;
double ar = 0.0;
for (double x1 = r+f; x1 < R-t-f; x1 += g+2*r)
for (double y1 = r+f; y1 < R-t-f; y1 += g+2*r) {
  double x2 = x1 + g - 2*f;
  double y2 = y1 + g - 2*f;
  if (x2 <= x1 || y2 <= y1) continue;
  if (x1*x1 + y1*y1 >= rad*rad) continue;
  if (x2*x2 + y2*y2 <= rad*rad) {
    // All points are inside circle.
    ar += (x2-x1)*(y2-y1);
  } else if (x1*x1 + y2*y2 >= rad*rad &&
             x2*x2 + y1*y1 >= rad*rad) {
    // Only (x1,y1) inside circle.
    ar += circle_segment(rad, acos(x1/rad) - asin(y1/rad)) +
          (sqrt(rad*rad - x1*x1)-y1) *
          (sqrt(rad*rad - y1*y1)-x1) / 2;
  } else if (x1*x1 + y2*y2 >= rad*rad) {
    // (x1,y1) and (x2,y1) inside circle.
    ar += circle_segment(rad, acos(x1/rad) - acos(x2/rad)) +
          (x2-x1) * (sqrt(rad*rad - x1*x1)-y1 +
                     sqrt(rad*rad - x2*x2)-y1) / 2;
  } else if (x2*x2 + y1*y1 >= rad*rad) {
    // (x1,y1) and (x1,y2) inside circle.
    ar += circle_segment(rad, asin(y2/rad) - asin(y1/rad)) +
          (y2-y1) * (sqrt(rad*rad - y1*y1)-x1 +
                     sqrt(rad*rad - y2*y2)-x1) / 2;
  } else {
    // All except (x2,y2) inside circle.
    ar += circle_segment(rad, asin(y2/rad) - acos(x2/rad)) +
          (x2-x1)*(y2-y1) -
          (y2-sqrt(rad*rad - x2*x2)) *
          (x2-sqrt(rad*rad - y2*y2)) / 2;
  }
}
printf("Case #%d: %.6lf\n", prob++, 1.0 - ar / (PI*R*R/4));
```

This solution takes O(S2) time, where S is the number of vertical strings of the racquet. It's not hard to come up with an O(S) solution because there are at most 4S border squares which can be found efficiently, but the previous solution was fast enough.

Instead of solving the problem exactly, an iterative solution which approximates the area to the needed precision would have also worked. One such solution uses divide and conquer by splitting the square into four smaller squares and then checking the simple cases where the squares are totally inside or totally outside the square. In the cases where the circle and square intersect just recurse if the current square is larger than some chosen precision. An observation is that we can divide every length by the radius of the racquet because it gets canceled in the division between the area of the gaps in the racquet and the disc area. This observation helps the iterative solution since we can make the number of iterations smaller. Here's some sample code:

```c++
double intersection(double x1, double y1,
                    double x2, double y2) {
  // the normalized radius is 1
  if (x1*x1 + y1*y1 > 1) {
    return 0;
  }
  if (x2*x2 + y2*y2 < 1) {
    return (x2-x1) * (y2-y1);
  }
  // EPS2 = 1e-6 * 1e-6
  if ((x2-x1)*(y2-y1) < EPS2) {
    // this trick helps in doing 10 or 100 times less
    // iterations than we would need to get the same
    // precision if we just return 0;
    return (x2-x1) * (y2-y1) / 2;
  }
 
  double mx = (x1 + x2) / 2;
  double my = (y1 + y2) / 2;
 
  return intersection(x1, y1, mx, my) +
    intersection(mx, y1, x2, my) +
    intersection(x1, my, mx, y2) +
    intersection(mx, my, x2, y2);
}
```

### Resource.
https://codingcompetitions.withgoogle.com/codejam/round/0000000000432b79/0000000000432f32#problem