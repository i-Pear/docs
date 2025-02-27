
The goal of this exercise is to modify **Config.uk**, for the [helloworld](https://github.com/unikraft/app-helloworld) app, so that the user can choose if the app will display *Hello world*, or what it reads from the file, similar to what we did on the previous exercise.

First of all, make sure that you setup the [helloworld](https://github.com/unikraft/app-helloworld) application just like in the previous sessions.

To complete our task, we need to add a new configuration in `Config.uk`.
We will do it like this:

```console
config APPHELLOWORLD_READFILE
	bool "Read my file"
	default n
	help
	  Reads the file in guest_fs/ and prints its contents,
	  instead of printing helloworld
```

After this, we need to modify our code in `main.c`, to use this configuration option.

```c
#ifndef CONFIG_APPHELLOWORLD_READFILE
	printf("Hello world!\n");
#else
	FILE *in = fopen("file", "r");
	char buffer[100];

	fread(buffer, 1, 100, in);
	printf("File contents: %s\n", buffer);
	fclose(in);
#endif
```

Note that, for our configuration option `APPHELLOWORLD_READFILE`, a symbol, `CONFIG_APPHELLOWORLD_READFILE`, was defined.
We tell GCC that, if that symbol was not defined, it should use the `printf("Hello world!\n")`.
Otherwise, it should use the code written by us.

The last step is to configure the application.
We do this by running `make menuconfig`, then going to the `Application Options` and enabling our configuration option, very similar to previous exercise.

Now we can build and run the new Unikraft image.
