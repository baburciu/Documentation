# Maintain doc by [markdown syntax](https://www.markdownguide.org/basic-syntax/) & [tips](https://guides.github.com/features/mastering-markdown/)
 
## 0. How to format:
<br/>
 >  - to start the localhost auto-reloading webserver of http://127.0.0.1:8000<br/>
<br/>
[boburciu@r220 ~]$ `cd Work_documenting/KVM-notes-proj/ `  <br/>
[boburciu@r220 KVM-notes-proj]$ `mkdocs serve`<br/>
<br/>
 >  - for code blocks, in order to preserve newline: <br/>
  >>  1. select text, then `ALT+SHIFT+I`; <br/>
  >>  2. then add `"<br/>"` without spaces to the end on line;<br/>
<br/>
 >  - for section hints just add ">" and then "-"before the line.<br/>
<br/>
 >  - to highlight:<br/>
\*This text will be italic\*<br/>
\_This will also be italic\_<br/>
<br/>
\*\*This text will be bold\*\*<br/>
\_\_This will also be bold\_\_<br/>
<br/>
\_You \*\*can\*\* combine them\_<br/>
<br/>
 >  - to stop for interprating, escape chars with "\"

## I. How to add page to doc main page:

[boburciu@r220 ~]$ <br/>
[boburciu@r220 ~]$ cd Work_documenting/ <br/>
[boburciu@r220 Work_documenting]$ <br/>
[boburciu@r220 Work_documenting]$ sudo yum install mkdocs.noarch <br/>
: <br/>
Installed: <br/>
  mkdocs.noarch 0:0.14.0-10.el7 <br/>
 <br/>
Dependency Installed: <br/>
  fontawesome-fonts.noarch 0:4.1.0-2.el7 fontawesome-fonts-web.noarch 0:4.1.0-2.el7 python-tornado.x86_64 0:4.2.1-5.el7 <br/>
  python2-click.noarch 0:6.7-8.el7       python2-markdown.noarch 0:2.4.1-4.el7 <br/>
 <br/>
Complete! <br/>
[boburciu@r220 Work_documenting]$ mkdocs new KVM-notes-proj <br/>
[boburciu@r220 Work_documenting]$ <br/>
[boburciu@r220 Work_documenting]$ cd KVM-notes-proj <br/>
[boburciu@r220 KVM-notes-proj]$  <br/>
[boburciu@r220 KVM-notes-proj]$ pwd <br/>
/home/boburciu/Work_documenting/KVM-notes-proj <br/>
[boburciu@r220 KVM-notes-proj]$ ls -lt <br/>
total 4 <br/>
drwxrwxr-x. 2 boburciu boburciu 21 Nov 20 20:33 docs <br/>
-rw-rw-r--. 1 boburciu boburciu 19 Nov 20 20:33 mkdocs.yml <br/>
[boburciu@r220 KVM-notes-proj]$ ls -lt docs/ <br/>
total 4 <br/>
-rw-r--. 1 boburciu boburciu 484 Nov 20 20:33 index.md <br/>
[bobiu@r220 KVM-notes-proj]$ <br/>

 > download the HTML template and save it in .md file under projects's /docs/ (1)

[boburciu@r220 KVM-notes-proj]$ curl 'https://jaspervdj.be/lorem-markdownum/markdown.txt' > docs/about.md <br/>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current <br/>
                                 Dload  Upload   Total   Spent    Left  Speed <br/>
100 10    0  1810    0     0   2614      0 --:--:-- --:--:-- --:--:--  2615 <br/>
[bobiu@r220 KVM-notes-proj]$ <br/>

 > add the new .md file under project's mkdocs.yml (2)

[boburciu@r220 KVM-notes-proj]$ cat mkdocs.yml<br/>
site_name: My Docs<br/>
nav:<br/>
    - Home: index.md<br/>
    - About: about.md[boburciu@r220 KVM-notes-proj]$<br/>
[boburciu@r220 KVM-notes-proj]$<br/>
[boburciu@r220 KVM-notes-proj]$ cat mkdocs.yml<br/>
site_name: My Docs<br/>
nav:<br/>
    - Home: index.md<br/>
    - About: about.md<br/>
[boburciu@r220 KVM-notes-proj]$<br/>
[boburciu@r220 KVM-notes-proj]$<br/>

## III. How to commit to git repo

` git init ` <br/>
` git remote add doc https://github.com/bogdanadrian-burciu/Documentation.git ` <br/>
` git config --global credential.helper store ` <br/>
` git add . ` <br/>
` git commit -m "base setup"  ` <br/>
` git push --set-upstream doc master ` <br/>
` git status ` <br/>
` git push doc ` <br/>

## Further sections
