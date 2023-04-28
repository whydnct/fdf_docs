---
layout: default
title: Getting started
parent: MiniLibX
grand_parent: Libs
nav_order: 2
---

# Getting started
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Installation

### Compilation on macOS

Because MiniLibX requires Cocoa of MacOSX (AppKit) and OpenGL (it doesn't
use X11 anymore) we need to link them accordingly. This can cause a complicated
compilation process. A basic compilation process looks as follows.

For object files, you could add the following rule to your makefile, assuming
that you have the `mlx` source in a directory named `mlx` in the root of your
project:

```makefile
%.o: %.c
	$(CC) -Wall -Wextra -Werror -Imlx -c $< -o $@
```

To link with the required internal macOS API:

```makefile
$(NAME): $(OBJ)
	$(CC) $(OBJ) -Lmlx -lmlx -framework OpenGL -framework AppKit -o $(NAME)
```

Do mind that you need the `libmlx.dylib` in the same directory as your build
target as it is a dynamic library!

### Compilation on Linux

In case of Linux, you can use the [Codam provided zip](https://github.com/42Paris/minilibx-linux) which is a Linux
compatible MLX version. It has the exact same functions and shares the same
function calls. Do mind, that using memory magic on images can differ as object
implementations are architecture specific. Next, you should unzip the MLX
for Linux in a new folder, in the root of your project, called `mlx_linux`.

MiniLibX for Linux requires `xorg`, `x11` and `zlib`, therefore you will need to
install the following dependencies: `xorg`, `libxext-dev` and `zlib1g-dev`.
Installing these dependencies on Ubuntu can be done as follows:
```sh
sudo apt-get update && sudo apt-get install xorg libxext-dev zlib1g-dev libbsd-dev
```
Now all thats left is to configure MLX, just run the `configure` script in the
root of the given repository, and you are good to go.

For object files, you could add the following rule to your makefile, assuming
that you have the `mlx` for linux source in a directory named `mlx_linux` in the
root of your project:

```makefile
%.o: %.c
	$(CC) -Wall -Wextra -Werror -I/usr/include -Imlx_linux -O3 -c $< -o $@
```

To link with the required internal Linux API:

```makefile
$(NAME): $(OBJ)
	$(CC) $(OBJ) -Lmlx_linux -lmlx_Linux -L/usr/lib -Imlx_linux -lXext -lX11 -lm -lz -o $(NAME)
```

## Initialization

- include the `<mlx.h>` header to access all the functions
- execute `mlx_init` function.
	- to establish a connection to the correct graphical system
	- returns a `void *` which holds the location of our current MLX instance.
	- To initialize MiniLibX one could do the following:

```c
#include <mlx.h>

int	main(void)
{
	void	*mlx;

	mlx = mlx_init();
}
```

- You see nothing, cause you're not creating a window, just a mlx instance.
- create a window by calling `mlx_new_window`.
	- returns a `void *` to the window.
		- We can give the window height, width and a title.
- Window rendering, call `mlx_loop`.
- Let's create a window with a width of 1920, a height of 1080 and a name of "Hello world!":

```c
#include <mlx.h>

int	main(void)
{
	void	*mlx;
	void	*mlx_win;

	mlx = mlx_init();
	mlx_win = mlx_new_window(mlx, 1920, 1080, "Hello world!");
	mlx_loop(mlx);
}
```

## Writing pixels to a image

- `mlx_pixel_put` function is very, very slow.
	- Because it tries to push the pixel instantly to the window (without waiting for the frame to be entirely rendered).
	- Because of this, we will have to buffer all of our pixels to a image, which we will then push to the window.
- passing an image to `mlx`requires the following pointers:
	- `bpp`: bits per pixel.
		- pixels are ints, so usually 4 bytes
			- unless we are dealing with a small endian (most likely a remote display with 8 bit colors)
- Now we can initialize the image with size 1920×1080 as follows:

```c
#include <mlx.h>

int	main(void)
{
	void	*mlx;
	void	*img;

	mlx = mlx_init();
	img = mlx_new_image(mlx, 1920, 1080);
}
```

- we have an image. We need the memory address whose bits we're gonna mutate so we can write pixels.
	- We get that address with:

```c
#include <mlx.h>

typedef struct	s_data {
	void	*img;
	char	*addr;
	int		bits_per_pixel;
	int		line_length;
	int		endian;
}				t_data;

int	main(void)
{
	void	*mlx;
	t_data	img;

	mlx = mlx_init();
	img.img = mlx_new_image(mlx, 1920, 1080);

	/*
	** After creating an image, we can call `mlx_get_data_addr`, we pass
	** `bits_per_pixel`, `line_length`, and `endian` by reference. These will
	** then be set accordingly for the *current* data address.
	*/
	img.addr = mlx_get_data_addr(img.img, &img.bits_per_pixel, &img.line_length,
								&img.endian);
}
```

- `bits_per_pixel`, `line_length` and `endian` are set by MiniLibX.

- Now we have the image address, but still no pixels.
- Before we start with this, we must understand that the bytes are not aligned, == the `line_length` differs from the actual window width.
	- So we should ALWAYS calculate the `memory offset` using the line length set by `mlx_get_data_addr`.

```c
int offset = (y * line_length + x * (bits_per_pixel / 8));
```

- Now that we know where to write, we can write a function like `mlx_pixel_put` but much faster:

```c
typedef struct	s_data {
	void	*img;
	char	*addr;
	int		bits_per_pixel;
	int		line_length;
	int		endian;
}				t_data;

void	my_mlx_pixel_put(t_data *data, int x, int y, int color)
{
	char	*dst;

	dst = data->addr + (y * data->line_length + x * (data->bits_per_pixel / 8));
	*(unsigned int*)dst = color;
}
```

- Screen-tearing.
	- Caused by changing an image represented in real time in a window.
	- You create 2 or more images to hold the frames temporarily.
		- You write them to a temporary image so you dont have to write to the presented image.

## Pushing images to a window

- Image created
	- we can push it to a window so that we can see it.

```c
#include <mlx.h>
// Example: write a pixel at (5,5) and put it to the window
typedef struct	s_data {
	void	*img;
	char	*addr;
	int		bits_per_pixel;
	int		line_length;
	int		endian;
}				t_data;

int	main(void)
{
	void	*mlx;
	void	*mlx_win;
	t_data	img;

	mlx = mlx_init();
	mlx_win = mlx_new_window(mlx, 1920, 1080, "Hello world!");
	img.img = mlx_new_image(mlx, 1920, 1080);
	img.addr = mlx_get_data_addr(img.img, &img.bits_per_pixel, &img.line_length,
								&img.endian);
	my_mlx_pixel_put(&img, 5, 5, 0x00FF0000);
	mlx_put_image_to_window(mlx, mlx_win, img.img, 0, 0);
	mlx_loop(mlx);
}
```

Note that `0x00FF0000` is the hex representation of `ARGB(0,255,0,0)`.

## Test your skills!

Now you that you understand the basics, get comfortable with the library and do some funky stuff! Here are a few ideas:
- Print squares, circles, triangles and hexagons on the screen by writing the pixels accordingly.
- Try adding gradients, making rainbows, and get comfortable with using the rgb colors.
- Try making textures by generating the image in loops.
