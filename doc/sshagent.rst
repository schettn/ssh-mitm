SSH Agent
=========

There are several ways in which SSH keys can be managed locally. One of the most common use cases is to store a key in the file system. SSH clients are able to read them from specific directories. For example, an RSA key may be stored as .ssh/id_rsa in the user's home directory.

To protect these keys from unauthorized access after theft or loss, it is recommended to store them encrypted. For this purpose it is necessary to enter a password.

The SSH Agent can be used to manage these keys. The password input, for decrypting is only necessary once during loading into the SSH Agent. All further cryptographic operations are then performed without the need to enter a password.

For the communication between SSH Agent and SSH Client a Unix socket is created and stored in a new subdirectory in /tmp. Because of this design, any user with appropriate privileges, such as the root user, is able to access and use this Unix socket.

For this reason, it is important that privileged users are trusted or that their accounts are not compromised.

To protect against misuse, a key can be secured with SSH-Askpass or a FIDO2 key. In both cases, user confirmation is required.

The big advantage of a FIDO2 key is that the confirmation is done via a separate hardware and cannot be compromised by a malware infected machine. SSH-Askpass is a software solution that can be bypassed by malware or an attacker who controls the victim's desktop.

For this reason, the use of a FIDO2 key is recommended over the use of SSH-Askpass.


SSH Agent Forwarding
--------------------

Many SSH clients offer the possibility to pass a local agent to a remote server. The corresponding protocol was defined in draft-ietf-secsh-agent-00. The corresponding draft was already defined in 2001 and almost all SSH clients support it.

A passed SSH agent can then be used to login to another server.

The advantage is that no sensitive data, such as private SSH keys, need to be stored permanently on the remote servers, but a secure login using Publickey authentication is still possible.

In most cases, agent forwarding is only supported for a shell connection. Agent forwarding is theoretically also possible for file transfers using SCP and SFTP, but most programs do not support this feature.

OpenSSH has implemented agent forwarding with version 8.4 for the client programs scp and sftp as well, in order to not have to copy these files via the local host for remote to remote file operations.

However, SSH Agent Forwarding is associated with a security risk. This is because privileged users can access and abuse the forwarded agent sockets.

For this reason, agent forwarding should not be used.

However, there are use cases where working without agent forwarding, is more costly. One possibility is working on a development server. From this server, it is often necessary to access a Git server to synchronize changes. Without a forwarded agent, custom keys would have to be created to access the Git server. These, in turn, could be stolen and thus abused if the server were compromised.

There is a tutorial on Github (https://docs.github.com/en/developers/overview/using-ssh-agent-forwarding) that describes how to configure OpenSSH to pass an agent through to a remote server.

However, it does not address the risk that a leaked agent is a potential security risk. The only warning is that the configuration must only be done for a specific host, otherwise the agent will be passed through to all servers you connect to.
In order to make it as difficult as possible to misuse the leaked keys, it is necessary to protect them with a FIDO2 token or SSH-Askpass. In the case of a passed-through agent, both solutions have a comparable level of security.

Nevertheless, the use of FIDO2 keys is recommended because a vulnerability in the client could eventually leak them. An example of this was the experimental support for roaming in OpenSSH 5.4. This feature should make it possible for a client to resume an unexpectedly terminated connection. Although the OpenSSH server did not support roaming, roaming was enabled in the client by default.

The roaming implementation had two vulnerabilities that allowed an attacker to access sensitive information such as private keys under certain circumstances.

.. warning::

    SSH Agent Forwarding should not be used. The reason is that it can prevent a lot of security risks. Agent forwarding often makes it easier to work with multiple servers. However, for most use cases there are ways to accomplish the same tasks without agent forwarding.

    If agent forwarding is still required, an FIDO2 token should be used. If this is not possible, e.g. because the server does not support the required algorithms, SSH-Askpass can also be used.
