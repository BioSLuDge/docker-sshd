#biosludge/sshd

Easy to use SSH and SFTP ([SSH File Transfer Protocol](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol)) server with [OpenSSH](https://en.wikipedia.org/wiki/OpenSSH). This is an automated build linked with the [debian](https://hub.docker.com/_/debian/) repository.

# Step 1 Usage

- Define users as command arguments, STDIN
  (syntax: `user:pass[:e][:uid[:gid]]...`).
  - You must set custom UID for your users if you want them to make changes to
    your mounted volumes with permissions matching your host filesystem.
- Mount volumes in user's home folder.
  - The users are chrooted to their home directory, so you must mount the
    volumes in separate directories inside the user's home directory
    (/home/user/**mounted-directory**).

# Step 3 Examples

## Docker Run

```
docker run \
    -v /share:/home/user/share \
    -p 2222:22 -d maltyxx/sshd \
    user:password:1001:1001
```

## Docker Compose

```
sshd:
    image: maltyxx/sshd
    volumes:
        - /share:/home/user/share
    ports:
        - "2222:22"
    command: user:password:1001:1001
```

# Step 6 Encrypted password

Add `:e` behind password to mark it as encrypted.

## Generate encrypted password

Tip: you can use makepasswd to generate encrypted passwords:

`echo -n "password" | makepasswd --crypt-md5 --clearfrom -`

## Docker Run

```
docker run \
    -v /share:/home/user/share \
    -p 2222:22 -d maltyxx/sshd \
    "user:password:e:1001:1001 user2:password2:e:1002:1002"
```

## Docker Compose

Tip: Remplace in the password encrypted `$` with `$$`.

```
sshd:
    image: maltyxx/sshd
    volumes:
        - /share:/home/user/share
    ports:
        - "2222:22"
    command: "user:password:e:1001:1001 user2:password2:e:1002:1002"
```

## User list
/ssh/sshd-users.conf:

```
user:password:e:1001:1001
user2:password2:e:1002:1002
```

### Docker Run

```
docker run \
    -v /ssh:/etc/ssh:ro \
    -v /share:/home/user/share \
    -p 2222:22 -d maltyxx/sshd \
    "user:password:e:1001:1001 user2:password2:e:1002:1002"
```

### Docker Compose

```
sshd:
    image: maltyxx/sshd
    volumes:
        - /ssh:/etc/ssh:ro
        - /share:/home/user/share
    ports:
        - "2222:22"
```

# Step 7 Using SSH key (without password)

Mount all public keys in the user's `.ssh/keys/` folder. All keys are automatically
appended to `.ssh/authorized_keys`.

```
docker run \
    -v /host/id_rsa.pub:/home/user1/.ssh/keys/id_rsa.pub:ro \
    -v /host/id_other.pub:/home/user2/.ssh/keys/id_other.pub:ro \
    -v /host/share1:/home/user1/share \
    -v /host/share2:/home/user2/share \
    -p 2222:22 -d maltyxx/sshd \
    user::1000
```

# Step 8 Logging in

The OpenSSH server runs by default on port 22, and in this example, we are
forwarding the container's port 22 to the host's port 2222. To log in with an
OpenSSH client, run: `sftp -P 2222 user@<host-ip>` or `ssh -p 2222 user@<host-ip>` or `scp -p 2222 user@<host-ip>`.

# Origin
Originally cloned from https://github.com/maltyxx/docker-sshd but I believe it originated from https://github.com/sandialabs/slycat