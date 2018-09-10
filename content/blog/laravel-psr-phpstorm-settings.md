+++
Tags = ["laravel", "phpstorm", "psr"]
content = "The best approximation to a Prettier-like experience for PHP.\n\nOpen PhpStorm **Settings/Preferences** and search for **Code Style** \n\n![Phpstorm > Preferences > Code Style](/uploads/phpstorm-preference-code-style.png \"Phpstorm > Preferences > Code Style\")\n\nSelect *PHP*"
date = "2018-09-10T14:40:06+01:00"
draft = true
title = "Laravel PSR PhpStorm settings"
undefined = "Laravel PSR-2 PhpStorm settings"

+++
PhpStorm has a built-in support for PSR-1/2 standards, it can easily be enabled in the Preferences menu.

Type **Code Style** in the search box if you can't see it

![](/uploads/phpstorm-preference-code-style.png)

Then select **PHP** from the sub-menu. Now, pratically hidden in the right upper corner, there's a link (button?) **Set from...**

![](/uploads/screencast 2018-09-10 15-43-38.gif)

It's unclear why such a widely used setting is a floating writing in the interface.

Select **Predefined Styles > PSR1/PSR2**

Click **Apply/OK**

We're only halfway through, this only enforce the selected code style when invoking **Code > Reformat Code** (or Alt + Cmd + L on a Mac), and it does absolutely nothing when you save a file. _This might be what you want, and your journey ends here._ 

If you want to format the code automatically every time you save a file, there's one more step.

You can define a Macro and remap the save action to a list of  commands, specifically Reformat Code and then Save, but once you do that, there's no easy path to disable the formatting temporarily for whatever reason, so I suggest to use a very popular plugin: [https://github.com/dubreuia/intellij-plugin-save-actions](https://github.com/dubreuia/intellij-plugin-save-actions "Save Action Plugin")