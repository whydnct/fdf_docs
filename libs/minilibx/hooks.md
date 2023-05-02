---
layout: default
title: Hooks
parent: MiniLibX
grand_parent: Libs
nav_order: 5
---

# Hooks
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

- *hooking*
	- techniques used to modify the behaviour of software
	- by intercepting:
		- function calls
		- messages
		- events.
	- Code that handles such intercepted elements is called a hook.

## Introduction

Hooking is used for many purposes, including debugging and extending functionality. Examples might include intercepting keyboard or mouse event messages before they reach an application, or intercepting operating system calls in order to monitor behavior or modify the function of an application or another component. It is also widely used in benchmarking programs, for example to measure frame rate in 3D games, where the output and input is done through hooking.

## Hooking into key events

```c
#include <mlx.h>
#include <stdio.h>

typedef struct	s_vars {
	void	*mlx;
	void	*win;
}				t_vars;

int	key_hook(int keycode, t_vars *vars)
{
	printf("Hello from key_hook!\n");
	return (0);
}

int	main(void)
{
	t_vars	vars;

	vars.mlx = mlx_init();
	vars.win = mlx_new_window(vars.mlx, 640, 480, "Hello world!");
	mlx_key_hook(vars.win, key_hook, &vars);
	mlx_loop(vars.mlx);
}
```

- We have now registered a function that will print a message whenever we press a key.
- We register a hook function with `mlx_key_hook`.
	- It calls `mlx_hook` with the appropriate X11 event types.

## Hooking into mouse events

<img align="right" src="res/mouse-schema.png">

```c
mlx_mouse_hook(vars.win, mouse_hook, &vars);
```
Mouse code for MacOS:
  - Left click: 1
  - Right click: 2
  - Middle click: 3
  - Scroll up: 4
  - Scroll down : 5  


## Test your skills!

Now that you have a faint idea of what hooks are, we will allow you to create a few of your own. Create hook handlers that whenever:
- a key is pressed, it will print the key code in the terminal.
- the mouse if moved, it will print the current position of that mouse in the terminal.
- a mouse is pressed, it will print the angle at which it moved over the window to the terminal.


