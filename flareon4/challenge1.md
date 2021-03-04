---
layout: default
permalink: /flareon4/challenge1/
title: Challenge 1 login.html
---

[Go Back to All Challenges](https://nobarxtx.github.io/flareon4)

# Challenge 1: login.html #


Here is the contents of **login.html**

```
<!DOCTYPE Html />
<html>
    <head>
        <title>FLARE On 2017</title>
    </head>
    <body>
        <input type="text" name="flag" id="flag" value="Enter the flag" />
        <input type="button" id="prompt" value="Click to check the flag" />
        <script type="text/javascript">
            document.getElementById("prompt").onclick = function () {
                var flag = document.getElementById("flag").value;
                var rotFlag = flag.replace(/[a-zA-Z]/g, function(c){return String.fromCharCode((c <= "Z" ? 90 : 122) >= (c = c.charCodeAt(0) + 13) ? c : c - 26);});
                if ("PyvragFvqrYbtvafNerRnfl@syner-ba.pbz" == rotFlag) {
                    alert("Correct flag!");
                } else {
                    alert("Incorrect flag, rot again");
                }
            }
        </script>
    </body>
</html>
```

What gives away the answer is the variable name `rotFlag` and addition `c.charCodeAt(0) + 13`. Rotation or ROT is an older and simple technique for hiding readable strings. It's just a simple substitution cipher commonly known as ROT13. ROT13 means the characters will rotate 13 places.

A quick solution is python's codec library for using ROT13.

```
import codecs
print codecs.encode('PyvragFvqrYbtvafNerRnfl@syner-ba.pbz', 'rot_13')
```

It's also good to note that all of flare-on challenges end in **@flare-on.com** this makes it easy to help you guess the output.


[Next -> Challenge 2](https://nobarxtx.github.io/flareon4/challenge2)
