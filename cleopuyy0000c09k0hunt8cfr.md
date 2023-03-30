# Linux String Substition (sed)

**Sed** is a computer program that is used to make changes to text files. It reads the file line by line and performs operations on each line, such as finding and replacing text, deleting lines that match a certain pattern, or adding or removing text from certain lines. Sed is often used together with other programs to perform more complex text-processing tasks. It is mostly used on Unix/Linux systems and can be run from the command line.

## Task

> There is some data on `Nautilus App Server 2` in `Stratos DC`. which needs to be altered in several of the files. On `Nautilus App Server 2`, alter the `/home/BSD.txt` file as per the details given below:
> 
> a. Delete all lines containing word `software` and save results in `/home/BSD_DELETE.txt` file. (Please be aware of case sensitivity)
> 
> b. Replace all occurrences of the word `or` to `is` and save results in `/home/BSD_REPLACE.txt` file.
> 
> `Note:` Let's say you are asked to replace the word `to` with `from`. In that case, make sure not to alter any words containing this string; for example `upto`, `contributor`, etc.

## Information

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677294685541/587837be-e9f9-4f69-a1b1-f91dcd6bd99e.png align="left")

## Implementation

1. `ssh` into App Server 2 and enter the password of **BigGr33n** when prompted.
    
    ```bash
    ssh steve@172.16.238.11
    ```
    
2. Delete all lines containing word `software` and save results in `/home/BSD_DELETE.txt` file. (Please be aware of case sensitivity):
    
    ```bash
    sed '/software/d' BSD.txt > BSD_DELETE.txt
    ```
    
    This command uses **sed** to modify the contents of `BSD.txt` and write the results to a new file called `BSD_DELETE`. Specifically, the command does the following:
    
    1. `/software/d` is a command that specifies a pattern to match and an action to perform. In this case, the pattern is the string "software", and the action is to **delete** any line that contains that string.
        
    2. `BSD.txt` is the input file that sed should read and apply the command
        
    3. `>` is a redirection operator that tells the shell to redirect the output of the command to a new file instead of the screen.
        
    4. `BSD_REPLACE.txt` is the name of the file that sed should write the modified output to.
        
    
    \*\*\*Please be aware of case sensitivity\*\*\* is already taken care of here as sed by default is case sensitive and will match the exact case in the pattern. To make it case insensitive, i.e match both **Software**, **software**, **SOFTWARE,** etc, we can add the `I` flag to the action: `/software/dI`.
    
3. Replace all occurrences of the word `or` to `is` and save results in `/home/BSD_REPLACE.txt` file.
    
    ```bash
    sed 's_\bor\b_is_g' BSD.txt > BSD_REPLACE.txt
    ```
    
    1. `s` is a sed command that stands for "substitute". It tells sed to search for a pattern and replace it with something else.
        
    2. `_\bor\b_is` is the search pattern that sed will look for in the input text. This pattern consists of the following components:
        
        * `_` is a delimiter character that separates the search pattern from the replacement text. It can be any character, but in this case, it is used instead of the more common `/` character, to avoid conflicts with the `/` characters in the pattern.
            
        * `\b` is a regular expression metacharacter that matches a word **boundary**, i.e. the transition between a word character (alphanumeric or underscore) and a non-word character (such as space or punctuation). The word boundary ensures that only the exact word between the boundaries will be matched.
            
        * `or` is the exact string that sed will look for in the input text. Only whole words that match the pattern will be replaced, not substrings that contain "or", such as "for", "gore", etc.
            
        * `is` is the replacement text that sed will substitute for the matched pattern. In this case, the replacement text is simply the word "is".
            
    3. `g` is an option that stands for "global". It tells sed to perform the substitution operation globally, i.e. on all occurrences of the pattern in each line of the input text, rather than just the first occurrence.
        

### Learning Takeaways

1. There are a lot of differences between **sed** on macOS and **gnu-sed** used in Linux distros. For example, for this sample text, I had to the command substitution command differently:
    
    ```bash
    # me.md
    hello world
    goodbye world
    World Empire
    them form forth for maybe
    angry sleight for fore
    ```
    
    0n macOS, notice how I had to write the word boundaries:
    
    ```bash
    sed 's/[[:<:]]for[[:>:]]/pour/g' me.md
    
    hello world
    goodbye world
    World Empire
    them form forth pour maybe
    angry sleight pour fore
    ```
    
2. Brew has a `gnu-sed` formula that can be run on macOS. This to me is a better option than learning two different commands' quirks:
    
    ```bash
    brew install gnu-sed
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677615452777/31c1cb44-bfdc-4e92-ac8e-408f14b3813a.png)
    
    I will replace my default **sed** with **gsed** with the approach above. In my `.zshrc` file I can add the above **PATH** variable and then source the file:
    
    ```bash
    vim ~/.zshrc
    
    # replace gsed (gnu-sed from homebrew) with native sed
    export PATH="/opt/homebrew/opt/gnu-sed/libexec/gnubin:$PATH"
    
    source ~/.zshrc
    
    sed --version
    
    sed (GNU sed) 4.9
    Copyright (C) 2022 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
    
    Written by Jay Fenlason, Tom Lord, Ken Pizzini,
    Paolo Bonzini, Jim Meyering, and Assaf Gordon.
    
    This sed program was built without SELinux support.
    
    GNU sed home page: <https://www.gnu.org/software/sed/>.
    General help using GNU software: <https://www.gnu.org/gethelp/>.
    E-mail bug reports to: <bug-sed@gnu.org>.
    ```