# Installing a Comingle Server

These instructions on based on the more complicated
[installation instructions for Coauthor](https://github.com/edemaine/coauthor/blob/master/INSTALL.md).

## Test Server

Here is how to get a **local test server** running:

1. **[Install Meteor](https://www.meteor.com/install):**
   `curl https://install.meteor.com/ | sh` on UNIX,
   `choco install meteor` on Windows (in administrator command prompt
   after [installing Chocolatey](https://chocolatey.org/install))
2. **Download Comingle:** `git clone https://github.com/edemaine/comingle.git`
3. **Run meteor:**
   * `cd comingle`
   * `meteor npm install`
   * `meteor`

Even a test server will be accessible from the rest of the Internet,
on port 3000.

## Public Server

To deploy to a **public server**, we recommend deploying from a development
machine via [meteor-up](https://github.com/kadirahq/meteor-up).
Installation instructions:

1. Install Meteor and download Comingle as above.
2. Install `mup` via `npm install -g mup`
   (after installing [Node](https://nodejs.org/en/) and thus NPM).
3. Edit `.deploy/mup.js` to point to your SSH key (for accessing the server),
   and your SSL certificate (for an https server).
4. `cd .deploy`
5. `mup setup` to install all necessary software on the server
6. `mup deploy` each time you want to deploy code to server
   (initially and after each `git pull`)

## Configuration

[`Config.coffee`](Config.coffee) stores a few configuration options defining
Comingle's behavior:

* `newMeetingRooms` specifies rooms (including titles and templates)
  to automatically create in any newly created meeting.
* `defaultServers` specifies the default servers for Cocreate and Jitsi.

## Zoom Support

To use the [Zoom Web Client SDK](https://github.com/zoom/sample-app-web),
you need to sign up for an API Key &amp; Secret.  Go to the
[Zoom Marketplace](https://marketplace.zoom.us/) and select "Create a JWT App".
See https://marketplace.zoom.us/docs/sdk/native-sdks/web/getting-started/integrate

Then add the API Key &amp; Secret into [`settings.json`](settings.json)
at the root of this repository.  It should look something like this:

```json
{
  "zoom": {
    "apiKey": "YOUR_API_KEY",
    "apiSecret": "YOUR_API_SECRET"
  }
}
```

DO NOT commit your changes to this file into Git; the secret needs to
STAY SECRET.  To ensure Git doesn't accidentally commit your changes, use

```
git update-index --assume-unchanged settings.json
```

If you're deploying a public server via `mup`, it should pick up these keys.
If you're developing on a local test server, use the following instead of
`meteor`:

```
meteor --settings settings.json
```

## Application Performance Management (APM)

To monitor server performance, you can use one of the following:

* [Monti APM](https://montiapm.com/)
  (no setup required, free for 8-hour retention); or
* deploy your own
  [open-source Kadira server](https://github.com/kadira-open/kadira-server).
  To get this running (on a different machine), I recommend
  [kadira-compose](https://github.com/edemaine/kadira-compose).

After creating an application on one of the servers above,
edit `settings.json` to look like the following
(omit `endpoint` if you're using Monti):

```json
{
  "kadira": {
    "appId": "xxxxxxxxxxxxxxxxx",
    "appSecret": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "endpoint": "https://your-kadira-server:22022"
  }
}
```

If you want Zoom support and APM, `settings.json` should look like this
(omit `endpoint` if you're using Monti):

```json
{
  "zoom": {
    "apiKey": "YOUR_API_KEY",
    "apiSecret": "YOUR_API_SECRET"
  },
  "kadira": {
    "appId": "xxxxxxxxxxxxxxxxx",
    "appSecret": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "endpoint": "https://your-kadira-server:22022"
  }
}
```

## MongoDB

All of Comingle's data is stored in the Mongo database
(which is part of Meteor).
You probably want to do regular (e.g. daily) dump backups.
<!--
There's a script in `.backup` that I use to dump the database,
copy to the development machine, and upload to Dropbox or other cloud storage
via [rclone](https://rclone.org/).
-->

`mup`'s MongoDB stores data in `/var/lib/mongodb`.  MongoDB prefers an XFS
filesystem, so you might want to
[create an XFS filesystem](http://ask.xmodulo.com/create-mount-xfs-file-system-linux.html)
and mount or link it there.
(For example, I have mounted an XFS volume at `/data` and linked via
`ln -s /data/mongodb /var/lib/mongodb`).

`mup` also, by default, makes the MongoDB accessible to any user on the
deployed machine.  This is a security hole: make sure that there aren't any
user accounts on the deployed machine.
But it is also useful for manual database inspection and/or manipulation.
[Install MongoDB client
tools](https://docs.mongodb.com/manual/administration/install-community/),
run `mongo comingle` (or `mongo` then `use comingle`) and you can directly
query or update the collections.  (Start with `show collections`, then
e.g. `db.messages.find()`.)
On a test server, you can run `meteor mongo` to get the same interface.

## bcrypt on Windows

To install `bcrypt` on Windows (to avoid warnings about it missing), install
[windows-build-tools](https://www.npmjs.com/package/windows-build-tools)
via `npm install --global --production windows-build-tools`, and
then run `meteor npm install bcrypt`.
