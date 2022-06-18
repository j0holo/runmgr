# Runmgr

A client and server application where the server provides a web UI to
manage which service should run on which server (managed by the client
application).

The server sends the states to the clients and the clients
enforce the states.

The clients need some sort of artifact server which contains the binary of
the services it manages. This could be an S3 bucket. init files for the
services can be contained in the commands or maybe stored in a key-value
store on the client to check every t seconds if the file still matches.
Just as environment variables and config files. Possibly just push everything
to S3. Then S3 is the authentication to get things like configs etc.

I guess it would be optimal if the config files that are being managed are 
typed in Go. So an init file or a `/etc/login.conf.d/` file. In combination 
with various CLI tools which can validate their config files it should be a 
secure way to manage configuration files.

Most files and binaries can be easily hashed and compared. So the server can 
ask the client what version of the file it has right now.

## TODO

- Create a list of files I want to manage
- Create a type for each file I want to manage
- Figure out what RPC calls we need (fetch binary, send config, etc.)
- Who is the server and who is the client? Polling or sending?
- Does the client of server need to hold state?
  - The computer we manage need to hold some state I guess.
  - We can use boltDB to store various items in buckets (binaries, configs, 
    init files)
- The servers being managed can work asynchronously. Manager sends status 
  changes. Status changes get applied in the database. A background work 
  looks if it needs to update files or restart services.
- Do files that hold secrets need to be encrypted when pushed to S3?
  - How would the server and client share the same decryption key? Pub/Pirvate key
    - Do we still need something like SSH or Ansible to distribute these keys?
  - What options are available in Golang?
