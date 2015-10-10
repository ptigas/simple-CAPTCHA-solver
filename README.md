# Simple CAPTCHA solver in python

## Disclaimer

This is a simple solver for a very specific and easy-to-solve CAPTCHAs like those proposed [here](http://www.white-hat-web-design.co.uk/articles/php-captcha.php) and [here](http://www.white-hat-web-design.co.uk/articles/php-captcha.php). Use the force to find something more complex.

## The idea
In this example we are going to use the following images.

![](http://ptigas.com/blog/wp-content/uploads/2011/02/test1.jpg "test") 

![](http://ptigas.com/blog/wp-content/uploads/2011/02/test2.jpg "test2")

First of all, a fixed size (monospace) font has been used. This makes extracting all the letters and using them as masks to check each digit, one by one, very easy. Also, the alphabet is simple lowercase hexadecimal letters. Thus, we had to extract only 16 letters.

The first part was to to extract all the letters. To achieve that, first of all we sampled several images so as to be sure that the images we have contains all the 16 letters. Then, using a simple image editor we cropped all the letters, one by one. We had to be careful so all the letters be aligned properly. Here is the final mask.

![](http://ptigas.com/blog/wp-content/uploads/2011/02/letters.jpg "letters")

Now we can focus on the CAPTCHA. As you can notice there is some noise which we have to remove (lines and stuff). After playing with several techniques we finally ended to the following. We turned the image to greyscale. Then we used a threshold to remove some of the noise. Here is the example after the filtering (cropping also applied).

![](http://ptigas.com/blog/wp-content/uploads/2011/02/source.jpg "source")

So, now we have the image almost cleared and some letters to play with.

## Procedure

Move each letter across the image and take the difference of the pixels for each position and sum them. Thus for each position we have a score of how much the letter (mask) fits the letter behind it. Then, store for each letter the position where the maximum score found. Then sort by score, take the top five results (our captcha is five letters) and finally sort by position. The result is the CAPTCHA text.

More formally

Let

![d(I,l,o)=\sum_{0\leq i \leq W \\ 0 \leq j \leq H}{[I(o+i, j)-l(i,j)]}](http://s0.wp.com/latex.php?latex=d%28I%2Cl%2Co%29%3D%5Csum_%7B0%5Cleq+i+%5Cleq+W+%5C%5C+0+%5Cleq+j+%5Cleq+H%7D%7B%5BI%28o%2Bi%2C+j%29-l%28i%2Cj%29%5D%7D&bg=T&fg=333333&s=0 "d(I,l,o)=\sum_{0\leq i \leq W \\ 0 \leq j \leq H}{[I(o+i, j)-l(i,j)]}")

be the distance of the image ![I](http://s0.wp.com/latex.php?latex=I&bg=T&fg=333333&s=0 "I"), with the letter ![l](http://s0.wp.com/latex.php?latex=l&bg=T&fg=333333&s=0 "l") in position ![o](http://s0.wp.com/latex.php?latex=o&bg=T&fg=333333&s=0 "o")

Then

![p(I,l) = \arg\max_{o}d(I,l,o)](http://s0.wp.com/latex.php?latex=p%28I%2Cl%29+%3D+%5Carg%5Cmax_%7Bo%7Dd%28I%2Cl%2Co%29&bg=T&fg=333333&s=0 "p(I,l) = \arg\max_{o}d(I,l,o)")

Thus, we need 5 letters ![l_{1},l_{2},l_{3},l_{4},l_{5}](http://s0.wp.com/latex.php?latex=l_%7B1%7D%2Cl_%7B2%7D%2Cl_%7B3%7D%2Cl_%7B4%7D%2Cl_%7B5%7D&bg=T&fg=333333&s=0 "l_{1},l_{2},l_{3},l_{4},l_{5}") with maximum ![d(l_{i},I,o)](http://s0.wp.com/latex.php?latex=d%28l_%7Bi%7D%2CI%2Co%29&bg=T&fg=333333&s=0 "d(l_{i},I,o)") ordered by ![p(l_{i}, I)](http://s0.wp.com/latex.php?latex=p%28l_%7Bi%7D%2C+I%29&bg=T&fg=333333&s=0 "p(l_{i}, I)").

```python
def p(img, letter):
    A = img.load()
    B = letter.load()
    mx = 1000000
    max_x = 0
    x = 0
    for x in xrange(img.size[0]-letter.size[0]):
        sum = 0
        for i in xrange(letter.size[0]):
            for j in xrange(letter.size[1]):
                sum = sum + abs(A[x+i, j][0] - B[i, j][0])
        if sum &lt; mx :
            mx = sum
            max_x = x
    return (mx, max_x)
```

Here is the code which implements this method. You can browse and download everything from [https://github.com/ptigas/simple-CAPTCHA-solver](https://github.com/ptigas/simple-CAPTCHA-solver)

```python
from PIL import Image

def ocr(im, threshold = 200, alphabet = "0123456789abcdef"):
    img = Image.open(im)
    img = img.convert("RGB")
    box = (8, 8, 58, 18)
    img = img.crop(box)
    pixdata = img.load()

    letters = Image.open('letters.bmp')
    ledata = letters.load()

    # Clean the background noise, if color != black, then set to white.
    for y in xrange(img.size[1]):
        for x in xrange(img.size[0]):
            if not(pixdata[x, y][0] &gt; threshold \
            and pixdata[x, y][1] &gt; threshold \
            and pixdata[x, y][2] &gt; threshold):
                pixdata[x, y] = (0, 0, 0, 255)
            else:
                pixdata[x, y] = (255, 255, 255, 255)

    counter = 0;
    old_x = -1;

    letterlist = []

    for x in xrange(letters.size[0]):
        black = True
        for y in xrange(letters.size[1]):
            if ledata[x, y][0] &lt;&gt; 0 :
                black = False
                break
        if black :
            if True :
                box = (old_x+1, 0, x, 10)
                letter = letters.crop(box)
                t = p(img, letter);
                print counter, x, t
                letterlist.append((t[0],alphabet[counter], t[1]))
            old_x = x
            counter = counter + 1

    box = (old_x+1, 0, 140, 10)
    letter = letters.crop(box)
    t = p(img, letter)
    letterlist.append((t[0],alphabet[counter], t[1]))

    t = sorted(letterlist)
    t = t[0:5] # 5-letter captcha

    final = sorted(t, key=lambda x: x[2])
    answer = ""
    for l in final:
        answer = answer + l[1]
    return answer

print ocr('test.jpg')
```

p.s. I found [this](http://www.wausita.com/captcha/). Very nice tutorial for CAPTCHA solving using python and vector space searching.
