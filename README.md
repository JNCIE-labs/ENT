## JNCIE-ENT Labs ##

### Purpose ###

This repository exists as a central location for labs written by people
during their quest for the JNCIE-ENT exam.  These labs exist in a
BSD-license compliant form.  Please see the license file for licensing
details and permissions.

### Contributing ###

If you wish to contribute, please open an issue as a pull request
against this repo in GitHub and title the issue appropriately.  
Enter a description of your lab and the request will be reviewed.

To ensure successful acceptance, please submit your lab with the
following:

 - A new directory specifically for your lab
 - PNG- or PDF-formatted lab diagram
 - Markdown-formatted lab exercise
 - Initial configuration files for each device
 - Solution configuration files for each device

At your discretion, you may include a walkthrough of the solution,
but this is not required.  You may also organize your subdirectory
however you wish, although it's recommended that your configuration
files remain in a subdirectory of your lab.

### Contribution Notice ###

> Any labs submitted must be compliant with this repositories license.
> Do NOT submit any labs for which you are not the original author or
> do not have the original author's express permission!

### Getting the Labs ###

At this time, as the labs are in a constant state of flux, the best
way to obtain them is with `git`.  While there are graphical clients
for `git`, this section only discusses the command-line method for
cloning this repo.

> This assumes you are using Linux or Mac OS X and already have `git`
> installed.  Further, this assumes you are familiar with using the
> terminal on your OS of choice.

To start, open your terminal application and create a new directory
for your `git` repos.  Skip the `mkdir` step below if you already
have such a directory.  Then, move into that directory.  Finally,
clone this repo into its own directory.

    cd ~
    mkdir git && cd $_
    git clone https://github.com/JNCIE-labs/ENT.git jncie-ent-labs
    
Now you have a current (as of the entry of those commands) version
of this repo.  To update the repo, use the `git pull` command, as
below.

    cd ~/git/jncie-ent-labs && git pull

Daily updates are recommended.  This will save you from constantly
checking for updates to the branch and manually pulling them.
On Linux and OS X, you can use `cron` to automatically perform a
specified command, series of commands, or script at specific
intervals.  To do this, you will need to create a `crontab` entry
and configure that entry to change to your directory for this repo
and issue the git pull command.  From your terminal application,
enter the following commands:
    
> This will open a crontab editor using your EDITOR environment
> variable.  This guide assumes your editor is `vi` or `vim`.
> If not, you probably know how to save and close the file.
> `nano` will tell you at the bottom of the screen.

    crontab -e
    i
    0 0 * * * cd ~/git/jncie-ent-labs && git pull
    [esc]
    :wq

### The Future ###

The organizers would like to see this become a full JNCIE-ENT lab
workbook for future candidates.  A community-driven, freely
available workbook could significantly aid candidates on a budget
or who might need to decide between which books are most vital.

> At this time, this repo is not ready to be considered a book
> or a replacement for any lab guide or workbook.  Although the
> plan is to get to that point, there is currently not enough
> material.  Please contribute if you'd like to see that change.

### The Organizers ###

The JNCIE-ENT Beta Lab Exam Study Group was founded by Tyler
Christiansen.  This repository was also founded by Tyler,
and Tim Hoffman is a leading contributor, particularly
to more advanced labs.

You can find Tyler on the Twitters [@packettalk][1] or by e-mail
at <tylerc@beatsmusic.com>.

You can find Tim on the Twitters [@hoffnz][2].

In addition, you can join the JNCIE study group [here][3].

[1]: http://twitter.com/packettalk "Tyler Christiansen"
[2]: http://twitter.com/hoffnz "Tim Hoffman"
[3]: https://groups.google.com/forum/#!forum/jncie-ent-beta "JNCIE-ENT Beta Study Group"
