# Maintain doc, syntax iarkdown: https://www.markdownguide.org/basic-syntax/
 0. How to format:
 > for code blocks, iorder to preserve newline:
    select text, then ALT+SHIT+I; <br/>
    then add "< br / >" without spaces to the end on line;

 > for section hints just add ">" and then "-"before the line.

## I. How to add page to doc main page:

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

[boburciu@r220 KVM-notes-proj]$ curl 'https://jaspervdj.be/lorem-markdownum/markdown.txt' > docs/about.md
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 10    0  1810    0     0   2614      0 --:--:-- --:--:-- --:--:--  2615
[bobiu@r220 KVM-notes-proj]$

 > add the new .md file under project's mkdocs.yml (2)

[boburciu@r220 KVM-notes-proj]$ cat mkdocs.yml
site_name: My Docs
nav:
    - Home: index.md
    - About: about.md[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$ cat mkdocs.yml
site_name: My Docs
nav:
    - Home: index.md
    - About: about.md
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$

## II. How to commit to git repo

Picto deus regis parantem foramina minuunt. Fessa quae, inexperrectus restabat
laterumque dabat gente **corpore**! In preces tectis aventi, et quos draconis
**et** forma muta anguis tonarent, haec algae a. Fervore fungi nutricibus visa
primi: ignes umbra quidem preces, silva Phoebe pennas.

Mores quod Alcides ante natum hic fulmen vivis cuti luce Eleusin coniurataeque
tempora relinquit inertem protinus condere claudor siquis. Pondere hunc? Argolis
ubi incaluit, sibi messibus compescuit unus iussus si vota promptior nonus mille
creavit, cum.

## Liberat hos aevi ut tamen Iliaden ventis

Senior hostisque *dominoque iuvencae dilexit* sperne, in oculis aequora, et
etiamnum moenia virtute ament modo baculum verba? Mors anilia ille Hylonome,
gestu **defessa** et sustinet arcum.

- Opem reddit mentis non mare pectora ducar
- Et pariter aditus
- Nomen tu usus
- Relictum domitae ad cursus inquit iamque illam
- Cecidisse prohibente
- Et in rupit poterat e remos cuiquam

Tauri servanda adeam! Et harundine dulces lunam: in surgit, stabula rarissima
fallaci times vestibus et. Ausim et tunc exstincta eripuit, creator sine, orbem?
Recondidit labor et [quae furoris quoque](http://labaret.io/), sonum, est sim
cuique.
