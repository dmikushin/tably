# A nice and clean looking table-like Git graph output for shells

[Published](https://stackoverflow.com/a/61487052/4063520) on StackOverflow by @onemorequestion and made into a GitHub repository under his permission.

[![With hashes as usually besides the graph tree][1]][1]

With hashes as usually besides the graph tree

[![Or in an extra column][2]][2]

Or in an extra column

**EDIT**: You want to start right away without reading all explanations? Jump to **EDIT 6**.

**INFO**: For a more branch-like colored version for shells, see also my second answer ([https://stackoverflow.com/a/63253135/][3]).

In all the answers to this question none showed a clean table-like looking output for shells so far.
The closest was [this answer from gospes][4] where I started from.

The core point in my approach is to count only the tree characters shown to the user. Then fill them to a personal length with white spaces.

Other than Git, you need these tools

 - grep
 - paste
 - printf
 - sed
 - seq
 - tr
 - wc

Mostly on board with any Linux distribution.

The code snippet is

    while IFS=+ read -r graph hash time branch message;do

      # Count needed amount of white spaces and create them
      whitespaces=$((9-$(sed -nl1000 'l' <<< "$graph" | grep -Eo '\\\\|\||\/|\ |\*|_' | wc -l)))
      whitespaces=$(seq -s' ' $whitespaces|tr -d '[:digit:]')

      # Show hashes besides the tree ...
      #graph_all="$graph_all$graph$(printf '%7s' "$hash")$whitespaces \n"

      # ... or in an own column
      graph_all="$graph_all$graph$whitespaces\n"
      hash_all="$hash_all$(printf '%7s' "$hash")  \n"

      # Format all other columns
      time_all="$time_all$(printf '%12s' "$time") \n"
      branch_all="$branch_all$(printf '%15s' "$branch")\n"
      message_all="$message_all$message\n"
    done < <(git log --all --graph --decorate=short --color --pretty=format:'+%C(bold 214)%<(7,trunc)%h%C(reset)+%C(dim white)%>(12,trunc)%cr%C(reset)+%C(214)%>(15,trunc)%d%C(reset)+%C(white)%s%C(reset)' && echo);

    # Paste the columns together and show the table-like output
    paste -d' ' <(echo -e "$time_all") <(echo -e "$branch_all") <(echo -e "$graph_all") <(echo -e "$hash_all") <(echo -e "$message_all")

To calculate the needed white spaces we use

      sed -nl1000 'l' <<< "$graph"

to get all characters (till 1000 per line) than select only the tree characters: * | / \ _ and white spaces with

      grep -Eo '\\\\|\||\/|\ |\*|_'

Finally count them and substract the result from our chosen length value, which is 9 in the example.

To produce the calculated amount of white spaces we use

      seq -s' ' $whitespaces

and truncate the position numbers with

      tr -d '[:digit:]'

Then add them to the end of our graph line. That's it!

Git has the nice option to [format the length of the output specifiers][5] already with the syntax `'%><(amount_of_characters,truncate_option)'`,
which adds white spaces from the left '>' or right '<' side and can truncate characters from the start 'ltrunc', middle 'mtrunc' or end 'trunc'.

It is **important** that printf cmd's above use the same length values for the corresponding Git column.

**Have fun to style your own clean table-like looking output to your needs.**

Extra:

To get the right length value you can use the following snippet

    while read -r graph;do
        chars=$(sed -nl1000 'l' <<< "$graph" | grep -Eo '\\\\|\||\/|\ |\*|_' | wc -l)
        [[ $chars -gt ${max_chars:-0} ]] && max_chars=$chars
    done < <(git log --all --graph --pretty=format:' ')

and use $max_chars as the right length value above.
<br>

**EDIT 1**:
Just noticed that the underline character is also used in the  git tree and edit the code snippets above accordingly. If there are other characters missing, please leave a comment.
<br>

**EDIT 2**:
If you want to get rid of the brackets around branch and tag entries, just use "%D" instead of "%d" in the git command, like in EDIT 3.
<br>

**EDIT 3**:
Maybe the "auto" color option is the one you prefer most for branch and tag entries?

[![Git bracketless auto color head and tag tablelike shell output][6]][6]

Change this part of the git command (color **214**)

    %C(214)%>(15,trunc)%D%C(reset)

to **auto**

    %C(auto)%>(15,trunc)%D%C(reset)
<br>

**EDIT 4**: Or you like your own color mix for that part, a fancy output with blinking head?

[![Git tree fancy styled tablelike output][7]][7]

To be able to style the head, branch names and tags first we need the "auto" color option in our git command like in EDIT 3.

Then we can replace the know color values with our own by adding these 3 lines

     # branch name styling
     branch=${branch//1;32m/38;5;214m}
     # head styling
     branch=${branch//1;36m/3;5;1;38;5;196m}
     # tag styling
     branch=${branch//1;33m/1;38;5;222m}

just before line

     branch_all="$branch_all$(printf '%15s' "$branch")\n"

in our code snippet. The replacement values produce the colors above.

For example the replacement value for head is

    3;5;1;38;5;196

where 3; stands for italic, 5; for blinking and 1;38;5;196 for the color. [For more infos start here.][8] Note: This behavior depends on your favorite terminal and may therefore not be usable.

**BUT** you can choose any color value you prefer.

**OVERVIEW of the git color values and ANSI equivalents**

[![enter image description here][9]][9]

You find a list with [git color/style option here][10].

If you need the output on your console for accurate colors (the picture above is scaled down by Stack Overflow) you can produce the output with

    for ((i=0;i<=255;i++));do
      while IFS='+' read -r tree hash;do
        echo -e "$(printf '%-10s' "(bold $i)") $hash  $(sed -nl500 'l' <<< "$hash"|grep -Eom 1 '[0-9;]*[0-9]m'|tr -d 'm')"
      done < <(git log --all --graph --decorate=short --color --pretty=format:'+%C(bold '$i')%h%C(reset)'|head -n 1)
    done

in your Git project path which uses the first commit from your Git log output.
<br>

**EDIT 5**:
As member "Andras Deak" mentioned, there are some ways how to use this code:

**1) as a Bash alias**:

[alias does not accept parameters but a function can][11], therefore just define in your .bashrc

       function git_tably () {
         unset branch_all graph_all hash_all message_all time_all max_chars

         ### add here the same code as under "2) as a shell script" ###

       }
and call the function git_tably (derived from table-like) directly under your git project path or from wherever you want with your git project path as first parameter.

**2) as a shell script**:

I use it with the option to pass a Git project directory as first parameter to it or if empty, take the working directory like the normal behavior. In it's entirety we have

    # Edit your color/style preferences here or use empty values for git auto style
    tag_style="1;38;5;222"
    head_style="1;3;5;1;38;5;196"
    branch_style="38;5;214"

    # Determine the max character length of your git tree
    while IFS=+ read -r graph;do
      chars_count=$(sed -nl1000 'l' <<< "$graph" | grep -Eo '\\\\|\||\/|\ |\*|_' | wc -l)
      [[ $chars_count -gt ${max_chars:-0} ]] && max_chars=$chars_count
    done < <(cd "${1:-"$PWD"}" && git log --all --graph --pretty=format:' ')

    # Create the columns for your preferred table-like git graph output
    while IFS=+ read -r graph hash time branch message;do

      # Count needed amount of white spaces and create them
      whitespaces=$(($max_chars-$(sed -nl1000 'l' <<< "$graph" | grep -Eo '\\\\|\||\/|\ |\*|_' | wc -l)))
      whitespaces=$(seq -s' ' $whitespaces|tr -d '[:digit:]')

      # Show hashes besides the tree ...
      #graph_all="$graph_all$graph$(printf '%7s' "$hash")$whitespaces \n"

      # ... or in an own column
      graph_all="$graph_all$graph$whitespaces\n"
      hash_all="$hash_all$(printf '%7s' "$hash")  \n"

      # Format all other columns
      time_all="$time_all$(printf '%12s' "$time") \n"
      branch=${branch//1;32m/${branch_style:-1;32}m}
      branch=${branch//1;36m/${head_style:-1;36}m}
      branch=${branch//1;33m/${tag_style:-1;33}m}
      branch_all="$branch_all$(printf '%15s' "$branch")\n"
      message_all="$message_all$message\n"

    done < <(cd "${1:-"$PWD"}" && git log --all --graph --decorate=short --color --pretty=format:'+%C(bold 214)%<(7,trunc)%h%C(reset)+%C(dim white)%>(12,trunc)%cr%C(reset)+%C(auto)%>(15,trunc)%D%C(reset)+%C(white)%s%C(reset)' && echo);

    # Paste the columns together and show the table-like output
    paste -d' ' <(echo -e "$time_all") <(echo -e "$branch_all") <(echo -e "$graph_all") <(echo -e "$hash_all") <(echo -e "$message_all")


**3) as a Git alias**:

Maybe the most comfortable way is to add a git alias in your .gitconfig

    [color "decorate"]
        HEAD = bold blink italic 196
        branch = 214
        tag = bold 222

    [alias]
        count-log = log --all --graph --pretty=format:' '
        tably-log = log --all --graph --decorate=short --color --pretty=format:'+%C(bold 214)%<(7,trunc)%h%C(reset)+%C(dim white)%>(12,trunc)%cr%C(reset)+%C(auto)%>(15,trunc)%D%C(reset)+%C(white)%s%C(reset)'
        tably     = !bash -c '"                                                                                                    \
                      while IFS=+ read -r graph;do                                                                                 \
                        chars_count=$(sed -nl1000 \"l\" <<< \"$graph\" | grep -Eo \"\\\\\\\\\\\\\\\\|\\||\\/|\\ |\\*|_\" | wc -l); \
                        [[ $chars_count -gt ${max_chars:-0} ]] && max_chars=$chars_count;                                          \
                      done < <(git count-log && echo);                                                                             \
                      while IFS=+ read -r graph hash time branch message;do                                                        \
                        chars=$(sed -nl1000 \"l\" <<< \"$graph\" | grep -Eo \"\\\\\\\\\\\\\\\\|\\||\\/|\\ |\\*|_\" | wc -l);       \
                        whitespaces=$(($max_chars-$chars));                                                                        \
                        whitespaces=$(seq -s\" \" $whitespaces|tr -d \"[:digit:]\");                                               \
                        graph_all=\"$graph_all$graph$whitespaces\n\";                                                              \
                        hash_all=\"$hash_all$(printf \"%7s\" \"$hash\")  \n\";                                                     \
                        time_all=\"$time_all$(printf \"%12s\" \"$time\") \n\";                                                     \
                        branch_all=\"$branch_all$(printf \"%15s\" \"$branch\")\n\";                                                \
                        message_all=\"$message_all$message\n\";                                                                    \
                      done < <(git tably-log && echo);                                                                             \
                      paste -d\" \" <(echo -e \"$time_all\") <(echo -e \"$branch_all\") <(echo -e \"$graph_all\")                  \
                                    <(echo -e \"$hash_all\") <(echo -e \"$message_all\");                                          \
                    '"

Than just call `git tably` under any project path.

Git is so powerful that you can change head, tags, ... directly as shown above and [taken from here][12].

[Another fancy option][13] is to select tree colors you prefer the most with

    [log]
        graphColors = bold 160, blink 231 bold 239, bold 166, bold black 214, bold green, bold 24, cyan

that gives you crazy looking but always table-like git log outputs

[![fanciest_git_tree_tablelike_image][14]][14]

Too much blinking! Just to demonstrate what is possible. Too few specified colors leads to color repetitions.

[A complete .gitconfig reference is just one click away.][15]
<br>

**EDIT 6**:
**Due to your positive votes I improved the snippet.
Now you can feed it with almost any git log command and don't have to adapt the code any more. Try it!**

**How does it work?**

 - define your Git log commands in your .gitconfig as always (formatted like below)
 - define a positive tree column number, where the git graph is shown (optional)

**Then just call**

&nbsp;&nbsp;&nbsp;&nbsp;`git tably YourLogAlias`

under any git project path or

&nbsp;&nbsp;&nbsp;&nbsp;`git tably YourLogAlias TreeColNumber`

where TreeColNumber overwrites an always defined value from above.

&nbsp;&nbsp;&nbsp;&nbsp;`git tably YourLogAlias | less -r`

will pipe the output into less which is useful for huge histories.
<br>
<br>
**Your Git log alias must follow these format rules:**

 - each column has to be indicated by a column delimiter which you have to choose and may cause problems if not unique

   i.e. `^` in `...format:'^%h^%cr^%s'` results in a tree, a hash, a time and a commit column
 - before every commit placeholder in your log command you have to use
   `%><(<N>[,ltrunc|mtrunc|trunc])` , with one of the trunc options

   (for syntax explanations see [https://git-scm.com/docs/pretty-formats][16]),

    however the last commit placeholder of any newline can be used without it

    i.e. `...format:'^%<(7,trunc)%h^%<(12,trunc)%cr^%s'`
 -  if extra characters are needed for decoration like `(committer: `, ` <` and `>)` in

    `...%C(dim white)(committer: %cn% <%ce>)%C(reset)...`

    to get a table-like output they must be written directly before and after the commit placeholder

    i.e. `...%C(dim white)%<(25,trunc)(committer: %cn%<(25,trunc) <%ce>)%C(reset)...`
 - using column colors like `%C(white)...%C(reset)` needs the `--color` option for a colored output

   i.e. `...--color...format:'^%C(white)%<(7,trunc)%h%C(reset)...`
 - if you use the `--stat` option or similar, add a newline `%n` at the end

   i.e. `...--stat...format:'...%n'...`
 - you can place the git graph at every column as long as you use no newline or only empty ones `format:'...%n'`

    for non-empty newlines `...%n%CommitPlaceholder...` you can place the git graph at every column n+1 only if all n-th columns of each line exist and use the same width

 - the name of your defined tree column number for a specific log alias have to be `YourLogAlias-col`

Compared to normal git log output this one is slow but nice.

**Now the improved snippet to add to your .gitconfig**

    [color "decorate"]
        HEAD   = bold blink italic 196
        branch = 214
        tag    = bold 222

    [alias]

        # Delimiter used in every mylog alias as column seperator
        delim     = ^

        # Short overview about the last hashes without graph
        mylog     = log --all --decorate=short --color --pretty=format:'^%C(dim white)%>(12,trunc)%cr%C(reset)^%C(bold 214)%<(7,trunc)%h%C(reset)' -5

        # Log with hashes besides graph tree
        mylog2    = log --all --graph --decorate=short --color --pretty=format:'%C(bold 214)%<(7,trunc)%h%C(reset)^%C(dim white)%>(12,trunc)%cr%C(reset)^%C(auto)%>(15,trunc)%D%C(reset)^%C(white)%<(80,trunc)%s%C(reset)'
        mylog2-col= 3

        # Log with hashes in an own column and more time data
        mylog3    = log --all --graph --decorate=short --color --pretty=format:'^%C(dim white)%>(12,trunc)%cr%C(reset)^%C(cyan)%<(10,trunc)%cs%C(reset)^%C(bold 214)%<(7,trunc)%h%C(reset)^%C(auto)%<(15,trunc)%D%C(reset)^%C(white)%s%C(reset)'
        mylog3-col= 4

        tably     = !bash -c '" \
                    \
                    \
                    declare -A col_length; \
                    apost=$(echo -e \"\\u0027\"); \
                    delim=$(git config alias.delim); \
                    git_log_cmd=$(git config alias.$1); \
                    git_tre_col=${2:-$(git config alias.$1-col)}; \
                    [[ -z "$git_tre_col" ]] && git_tre_col=1; \
                    [[ -z "$git_log_cmd" ]] && { git $1;exit; }; \
                    \
                    \
                    i=0; \
                    n=0; \
                    while IFS= read -r line;do \
                      ((n++)); \
                      while read -d\"$delim\" -r col_info;do \
                        ((i++)); \
                        [[ -z \"$col_info\" ]] && col_length[\"$n:$i\"]=${col_length[\"${last[$i]:-1}:$i\"]} && ((i--)) && continue; \
                        [[ $i -gt ${i_max:-0} ]] && i_max=$i; \
                        col_length[\"$n:$i\"]=$(grep -Eo \"\\([0-9]*,[lm]*trunc\\)\" <<< \"$col_info\" | grep -Eo \"[0-9]*\" | head -n 1); \
                        [[ -n \"${col_length[\"$n:$i\"]}\" ]] && last[$i]=$n; \
                        chars_extra=$(grep -Eo \"trunc\\).*\" <<< \"$col_info\"); \
                        chars_extra=${chars_extra#trunc)}; \
                        chars_begin=${chars_extra%%\\%*}; \
                        chars_extra=${chars_extra%$apost*}; \
                        chars_extra=${chars_extra#*\\%}; \
                        case \" ad aD ae aE ai aI al aL an aN ar as at b B cd cD ce cE ci cI cl cL cn cN cr \
                                cs ct d D e f G? gd gD ge gE GF GG GK gn gN GP gs GS GT h H N p P s S t T \" in \
                          *\" ${chars_extra:0:2} \"*) \
                            chars_extra=${chars_extra:2}; \
                            chars_after=${chars_extra%%\\%*}; \
                            ;; \
                          *\" ${chars_extra:0:1} \"*) \
                            chars_extra=${chars_extra:1}; \
                            chars_after=${chars_extra%%\\%*}; \
                            ;; \
                          *) \
                            echo \"No Placeholder found. Probably no tablelike output.\"; \
                            continue; \
                            ;; \
                        esac; \
                        if [[ -n \"$chars_begin$chars_after\" ]];then \
                          len_extra=$(echo \"$chars_begin$chars_after\" | wc -m); \
                          col_length["$n:$i"]=$((${col_length["$n:$i"]}+$len_extra-1)); \
                        fi; \
                      done <<< \"${line#*=format:}$delim\"; \
                      i=1; \
                    done <<< \"$(echo -e \"${git_log_cmd//\\%n/\\\\n}\")\"; \
                    \
                    \
                    git_log_fst_part=\"${git_log_cmd%%\"$apost\"*}\"; \
                    git_log_lst_part=\"${git_log_cmd##*\"$apost\"}\"; \
                    git_log_tre_part=\"${git_log_cmd%%\"$delim\"*}\"; \
                    git_log_tre_part=\"${git_log_tre_part##*\"$apost\"}\"; \
                    git_log_cmd_count=\"$git_log_fst_part$apost $git_log_tre_part$apost$git_log_lst_part\"; \
                    col_length[\"1:1\"]=$(eval git \"${git_log_cmd_count// --color}\" | wc -L); \
                    \
                    \
                    i=0; \
                    while IFS=\"$delim\" read -r graph rest;do \
                      ((i++)); \
                      graph_line[$i]=\"$graph\"; \
                    done < <(eval git \"${git_log_cmd/ --color}\" && echo); \
                    \
                    \
                    i=0; \
                    l=0; \
                    while IFS= read -r line;do \
                      c=0; \
                      ((i++)); \
                      ((l++)); \
                      [[ $l -gt $n ]] && l=1; \
                      while IFS= read -d\"$delim\" -r col_content;do \
                        ((c++)); \
                        [[ $c -le $git_tre_col ]] && c_corr=-1 || c_corr=0; \
                        if [[ $c -eq 1 ]];then \
                          [[ \"${col_content/\\*}\" = \"$col_content\" ]] && [[ $l -eq 1 ]] && l=$n; \
                          count=$(wc -L <<< \"${graph_line[$i]}\"); \
                          whitespaces=$(seq -s\" \" $((${col_length[\"1:1\"]}-$count))|tr -d \"[:digit:]\"); \
                          col_content[$git_tre_col]=\"${col_content}$whitespaces\"; \
                        else \
                          col_content[$c+$c_corr]=\"$(printf \"%-${col_length[\"$l:$c\"]}s\" \"${col_content:-\"\"}\")\"; \
                        fi; \
                      done <<< \"$line$delim\"; \
                      for ((k=$c+1;k<=$i_max;k++));do \
                        [[ $k -le $git_tre_col ]] && c_corr=-1 || c_corr=0; \
                        col_content[$k+$c_corr]=\"$(printf \"%-${col_length[\"$l:$k\"]:-${col_length[\"${last[$k]:-1}:$k\"]:-0}}s\" \"\")\"; \
                      done; \
                      unset col_content[0]; \
                      echo -e \"${col_content[*]}\"; \
                      unset col_content[*]; \
                    done < <(eval git \"$git_log_cmd\" && echo); \
                    "' "git-tably"

where in tably

 -  the first paragraph loads the delim(iter), YourLogAlias and YourLogAlias-col into shell variables
 -  the second reads out the length for each column
 -  the third counts the max. length of the tree
 -  the fourth loads the tree into an array
 -  the fifth organizes and print the table-like output

**Results:**

[![Enter image description here][17]][17]

[![Enter image description here][18]][18]

[![Enter image description here][19]][19]

or with new TreeColNumber on the fly

[![Enter image description here][20]][20]

**AGAIN: Have fun to style your own clean table-like looking output to your needs.**

**You can also share your preferred formatted Git log alias in the comments. From time to time I will include the most rated ones in the text above and add images too.**

  [1]: https://i.stack.imgur.com/ZLBab.png
  [2]: https://i.stack.imgur.com/Ta367.png
  [3]: https://stackoverflow.com/a/63253135/
  [4]: https://stackoverflow.com/a/22481650/8006273
  [5]: https://git-scm.com/docs/pretty-formats
  [6]: https://i.stack.imgur.com/kazSy.png
  [7]: https://i.stack.imgur.com/Zmfuc.gif
  [8]: https://stackoverflow.com/a/28938235
  [9]: https://i.stack.imgur.com/C6OIK.jpg
  [10]: https://stackoverflow.com/a/15458378
  [11]: https://stackoverflow.com/a/7131683/8006273
  [12]: https://stackoverflow.com/a/48695573
  [13]: https://stackoverflow.com/a/48695516
  [14]: https://i.stack.imgur.com/zlV4p.gif
  [15]: https://git-scm.com/docs/git-config
  [16]: https://git-scm.com/docs/pretty-formats#Documentation/pretty-formats.txt-emltltNgttruncltruncmtruncem
  [17]: https://i.stack.imgur.com/Uvs59.png
  [18]: https://i.stack.imgur.com/vdD7q.png
  [19]: https://i.stack.imgur.com/bHNY0.png
  [20]: https://i.stack.imgur.com/Wr4ec.png

