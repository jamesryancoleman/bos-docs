# Building on the deadband controller

Start by cloning this application to your working directory:

``` bash
cd ~/bos-apps
git clone https://github.com/jamesryancoleman/bos-app-deadband NEW_NAME
```

Update the contents of the env.spec.json with constraints relevant to your workflow. Notably, for autocomplete to work you want to specify the `preferred_class` and minimum `accept_class`. You MUST copy the contents over to the `bos.env.spec` `LABEL` as one line. In VS code you can compress to single line with `ctrl + j `. (and uncompress with `shift + alt + f`.)