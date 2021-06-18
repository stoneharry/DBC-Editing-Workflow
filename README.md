# DBC Editing Workflow
## Overview

The DBC file format does not lend itself to being edited directly very well. I would much rather have all the data stored in a SQL format so that we can perform queries and other operations on the data with ease. I am pretty sure that Blizzard will also have their data stored in another format and only convert it to DBC when it needs to be packaged.

When I was working on the [Spell Editor](https://github.com/stoneharry/WoW-Spell-Editor) I ended up writing code to convert DBC files to and from SQL with the use of text file 'Bindings' that define what the structure is for each DBC file. The program allows you to import and export any DBC file that you have a binding for. Using this functionality I was able to write a simple commnand line program called [HeadlessExporter](https://github.com/stoneharry/WoW-Spell-Editor/blob/master/HeadlessExport/Program.cs) that automatically connects to the database and exports all the Bindings to DBC files.

I then created a [Jenkins](https://www.jenkins.io/) automation job that when started:
- Runs the [HeadlessExporter](https://github.com/stoneharry/WoW-Spell-Editor/blob/master/HeadlessExport/Program.cs) program to export all the SQL tables to DBC files
- Moves all the exported DBC files to the server folder
- Moves all the DBC files to a client git repository
- Commits and pushes the DBC file changes in the client repo
- Restarts the worldserver

This means that I can make changes to the data on my own computer with a SQL editor, run the Jenkins job, pull the client repo (which automatically updates my WoW client folder with a symlink), and that's it - I can test it in game.

It has saved me a lot of time, I hope more people can make use of this.

## Jenkins job definition

Our Jenkins job definition is actually configured to also send a notification to Discord with a webhook.
```bash
cd C:\Users\Administrator\Desktop\Tools\HeadlessExport
call HeadlessExport.exe

cd C:\Users\Administrator\Desktop\ClientBuild

git reset --hard HEAD
git clean -f -d
git pull

xcopy /s /y C:\Users\Administrator\Desktop\Tools\HeadlessExport\Export C:\Users\Administrator\Desktop\ClientBuild\DBFilesClient

git add -A
git commit -m "Automatic Jenkins update of DBFilesClient"
git push

net stop "Dev World Server (managed by AlwaysUpService)"

xcopy /s /y "C:\Users\Administrator\Desktop\ClientBuild\DBFilesClient" "C:\HoT\Development\Server\dbc"

net start "Dev World Server (managed by AlwaysUpService)"
```

## Precompiled HeadlessExporter

There is a precompiled binary in this repository. You will need to edit the `config.xml` and get the Binding files from the Spell Editor repository. The `SpellEditor.exe` provided in this should not be used because it is an old version. I just never needed to update it since I compiled the `HeadlessExporter` a long time ago and it uses it as a reference for the DBC converting API.

## Example Queries
### Regenerate Item.dbc

You can regenerate your Item DBC from your server `item_template` data with one simple query. For example, for my TrinityCore world DB:
```sql
REPLACE INTO new_dbc.item
    SELECT entry, class, subclass, soundoverridesubclass, Material, displayid, InventoryType, sheath
    FROM new_world.item_template;
    -- WHERE entry BETWEEN 1 and 100; -- You could add a condition or just regenerate everything
```

## Screenshots

![Jenkins Build List](https://i.imgur.com/D1FgsmG.png)
![Data in MySQL](https://i.imgur.com/82V2IxE.png)
![Export Example 1](https://i.imgur.com/ZBtiSMe.png)
![Export Example 2](https://i.imgur.com/HfNhZop.png)
