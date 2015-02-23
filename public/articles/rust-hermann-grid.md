Happy inaugural post! Instead of just saying hi, why not jump into the project I worked on this morning: generating a Hermann Grid using Rust.
![Hermann Grid](/images/resources/hermann/hermann.jpeg)

A Hermann Grid is an optical illusion where dark patches appear between the corners of the squares you are not directly looking at. For more information, see [Michael Bach's excellent page on them](http://www.michaelbach.de/ot/lum_herGrid/index.html).

I liked the idea of having this optical illusion as my cover photo on social websites, but couldn't find a simple way to generate one. So instead of fumbling around in Inkskape until I made something passable, I built a little tool to generate one for me. So let's walk through the process.


## Installing Rust
First step, is to ensure you have a recent version of Rust set up. Most libraries are currently coding against nightly releases in lue of a 1.0 release. You can follow the instructions to install rust from the [Official Book](http://doc.rust-lang.org/book/installing-rust.html).

## Generating a project
Use Cargo to create a new bin project.
```bash
$ cargo new hermann
```

## Install dependencies
We're going to use Piston's image library to generate the image.
To add it to the project, add it to your `Cargo.toml`.
```yaml
[dependencies.image]

git = "https://github.com/PistonDevelopers/image"
```

## Generate an image
Creating images with the `image` library is pretty straightforward.
```rust
extern crate iamge;
use image::{ ImageBuffer, ImageRgb8, Rgb, JPEG };
use std::old_io::{ File };

fn gen() {
    let imgx = 150;
    let imgy = 50;
    let imgbuf = ImageBuffer::from_fn(imgx, imgy, |x, y| {
        Rgb([0, 0, 0])
    });
    let fpath = Path::new("herman.jpeg");
    let ref mut fout = File::create(&fpath).unwrap();
    let _ = ImageRgb8(imgbuf).save(fout, JPEG);
}
```
We create an `ImageBuffer` from a closure function, which returns an `Rgb` pixel for each coordinate.
Then we write the `ImageBuffer` out to a file using `ImageRgb8` and a `File`.
![Herman 1](/images/resources/hermann/hermann-1.jpeg)

## Hermannize the image
We can create a Hermann Grid using the pixel coordinates; splitting those into boxes by taking the remainder of the coordinate with the size of the box.
Let's make one with squares 8 pixels wide and margins 2 pixels wide.
```rust
fn gen() {
    let imgx = 150;
    let imgy = 50;

    let box_size = 10;
    let margin_low = 1;
    let margin_high = box_size - margin_low;

    let imgbuf = ImageBuffer::from_fn(imgx, imgy, |x, y| {
        let xx = x % box_size;
        let yy = y % box_size;

        if xx > margin_low && xx < margin_high &&
            yy > margin_low && yy < margin_high
        {
            Rgb([255, 255, 255])
        } else {
            Rgb([0, 0, 0])
        }
    });

    let fpath = Path::new("herman.jpeg");
    let ref mut fout = File::create(&fpath).unwrap();
    let _ = ImageRgb8(imgbuf).save(fout, JPEG);
}
```
![Herman 2](/images/resources/hermann/hermann-2.jpeg)

## Finishing up
Hooray! We have a Hermann Gird with the proper illusion effect. To make this useful for a banner image, we need to increase the resolution. The current guidelines for twitter are for an image about 1500px by 500px.

For a more complete version, check out [my GitHub repo](https://github.com/oppenlander/rust-hermann-generator). It contains some extra goodies, like the ability to invert the colors and choose the size.
