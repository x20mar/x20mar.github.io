---
title: "Fix Google Chrome with resetting your profile"
layout: "post"
---

Occasionally you may find that Google Chrome will suffer a crash, after which it will always display the error message

> Google Chrome was not shut down properly

Most documentation will tell you backup your profile folder and create a new one. But there is an easier way with a lot less hassle

* Go to your Google Profile folder:

```bash
C:\Users\NAME\AppData\Local\Google\Chrome\User Data\Default\
```

or

```bash
~/.config/google-chrome/Default
```

* Open the Preferences file. This contains a very large JSON object in here (don't be scared)
* Locate the key-pair

```JSON
"exit_type": "Crashed"
```

and change it to

```JSON
"exit_type": "normal"
```

Save the file and that's it. Chrome should stop harassing you.
