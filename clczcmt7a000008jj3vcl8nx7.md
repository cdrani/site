# Cat : A Bare-bones "Editor"

`cat` (i.e concatenate and print files)

I have used `cat` and most likely you have too, although my usage has mostly been limited to viewing files in the terminal, such as taking a glance at packages listed in a `package.json` or reading a `.env` file. The command has a lot of options, such as numbering lines, removing empty lines, etc. Take a look at some of them below and use the `man` pages to further explore.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673902419206/93ab8ff6-502c-4c1e-9fe5-760b23d1cd61.png align="center")

The special case which inspired this post is in the `DESCRIPTION` above:

> The cat utility reads files sequentially, `writing them to the standard output`. The file operands are processed in command-line order. If file is a single dash (‘-’) or absent, `cat reads from the standard input`.

`cat` can also ***read*** from standard input (STDIN), i.e keyboard input in our terminal. This means that it will regurgitate any input we enter back to the console if we don't supply a file to it from which to read.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673904197794/ca10fbc9-f9b4-4b19-bab7-308cad1f8a7b.png align="center")

Where's the editor part then? Well, instead of having `cat` writing out input to STDOUT (console), we can redirect the output to a file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673904521797/cdab160b-7887-4b3b-98b3-165a849dd5b8.png align="center")

A small caveat is that "editor" is somewhat of a misnomer here as while the program is running you do not have functionalities like editing or going back to a previous line, syntax highlighting, etc. This is sufficient enough in cases like above where one is maybe writing or appending to a todo-list, .gitignore, .env, etc. I have found this to be slightly more efficient than using `vim` to open/create, insert, save, and exit, or using `echo` with multi-line text requiring enclosing strings.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673905368722/9a2a1435-7efc-4109-bf19-7c60c5671b1b.png align="center")

I am going to be using this for a while and see if it sticks for the use cases above or any new one I discover.