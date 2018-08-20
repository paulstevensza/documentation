# Automating the Configuration of macOS

There's a repo I made which [configures a fresh macOS install](https://github.com/paulstevensza/bootstrap). It's written in
bash, and uses a series of Homebrew steps and direct commands to build a usable Mac without you having to sit there and
manually go through a few dozen steps to build and install software.

While I do do some ham fisted stuff, there are some pecularities to this OS that make automating it a pain in the ass. Some of
these are Apple-isms, others are the tools.

### 1. There's no git baked into the OS.

I mean, for real? It's 2018 for fuck sakes, and a large part of your target demographic are developers and other folk 
who've likely integrated git into at least some parts of their workflow. Granted, the first time you punch `git` into the
command line, it kicks off the XCode command line tools installation, but even this slimmed down toolset is a massive chunk
of software to digest before you can start doing anything.

### 2. Homebrew wants you to press enter to continue.

I've just pulled a random script off of the Internet, piped it through Ruby, and via `sudo`, given it the keys to the bloody
kingdom, and yet it sits there like an asshole (because *this* asshole does `> /dev/null 2>&1` to prettify output) waiting for
you to hit enter to continue. Given that a bad pipeline would allow someone to replace my bootscreen with dick pics if they
managed to get that into the repo unnoticed, I think that a canned message describing the things that Homebrew is about to do
is as useful as the license agreement on most software. Do I agree? Well, if I don't...I can't use my fucking computer now,
can I?

### 3. Seriously -- what's up with defaults.

There just has to be an easier way to configure settings on a damn OS. Is this because of plist files?

### 4. MAS say what now?

I thought that I'd authenticated my iCloud account when I set up my Mac (you kinda have to if that's where your backups live),
but I guess I forgot to explicitly tell the App Store that I am an authenticated user and that I might just want to use a
command line tool I found on the Internet to install Apps from a place that a) contains a bunch of information on me and
b) contains payment information to install my purchased apps. I can see why this would be seen as a bad idea: but it's **my**
bad idea damnit. Those YouTube likes aren't going to happen by themselves of I'm having to click around in the App Store
installing software that `mas` didn't.

## A new approach

I think I need a new approach to automating macOS configurations. This method works, but is a bit rough around the edges. I'm
thinking that I need to address the following:

1. Figure out a way to automatically tell Homebrew to go ahead.
2. Figure out whether I want to use Ansible or Chef to deploy the machine.
3. Build the machine.
4. Profit.

I do need to spend more time with a fresh Mac to test some theories. I hope that a bootable macOS thumb drive is still a
thing so that I don't punish my capped Internet at home.
