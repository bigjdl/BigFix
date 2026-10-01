# RepositorySiteExample
Example of a Repository Site, that could be gathered by BigFix 11.0.7 or later

This repository is intended to be an example, illustrating a possible directory structure layout for a Bigfix Repository Site, with example fixlets, and demonstrating use of siteConfig.xml, site.xml, and digest.xml

See Also:

[BigFix Repository Sites Documentation](https://help.hcl-software.com/bigfix/11.0/platform/Platform/Config/c_repository_site.html)

[BigFix Developer Site](https://developer.bigfix.com)

[BigFix Forum](https://forum.bigfix.com)


# Generating SSH Credential on BigFix Root Server

Open an Elevated Command Prompt
Navigate to the 'RepositorySiteGather\GitCredentials' directory beneath the BES Server directory, i.e.

```
CD "C:\Program Files (x86)\BigFix Enterprise\BES Server\RepositorySiteGather\GitCredentials"
```

Generate an SSH Public/Private Key Pair.  (As of 11.0.7, these specific filenames must be used):
```
# create id_rsa and id_rsa.pub
ssh-keygen -m PEM -t rsa -b 4096 -f id_rsa -N ""
```

Create the known_hosts file.  This example is for github.com, replace with your own git provider as necessary:
```
ssh-keyscan github.com >> known_hosts
```
**Note** : the ssh-keyscan from Windows may not be compatible and yield errors such as
```
 ssh-keyscan github.com >> known_hosts
# github.com:22 SSH-2.0-0e0c9ec
choose_kex: unsupported KEX method sntrup761x25519-sha512@openssh.com
# github.com:22 SSH-2.0-0e0c9ec
choose_kex: unsupported KEX method sntrup761x25519-sha512@openssh.com
# github.com:22 SSH-2.0-0e0c9ec
choose_kex: unsupported KEX method sntrup761x25519-sha512@openssh.com
# github.com:22 SSH-2.0-0e0c9ec
choose_kex: unsupported KEX method sntrup761x25519-sha512@openssh.com
# github.com:22 SSH-2.0-0e0c9ec
choose_kex: unsupported KEX method sntrup761x25519-sha512@openssh.com
```

In that case, install 'Git for Windows' client and use the openssh binaries from it:
```
"C:\Program Files\Git\usr\bin\ssh-keyscan.exe" github.com >> known_hosts
# github.com:22 SSH-2.0-0e0c9ec
# github.com:22 SSH-2.0-0e0c9ec
# github.com:22 SSH-2.0-0e0c9ec
# github.com:22 SSH-2.0-0e0c9ec
# github.com:22 SSH-2.0-0e0c9ec
```



Import the id_rsa.pub as an SSH Public Key on a github account as which your server will authenticate.  This account needs contents:read permission on any repository you wish to gather.

# Content Site Structure

Repositories must follow BigFix Support Site Propagation conventions:

An example directory structure may be illustrated as
```none
Fixlets/
   ├── Analyses/
   │    └─ 1300- Some Analysis.bes
   ├── Fixlets/
   │    └─ FixletType1/
   │    │     └─ digest.xml
   │    │     └─ 2110- Some Fixlet.bes
   │    └─ FixletType2/
   |          └─ 2210- Some Fixlet.bes
   └── Tasks/
        └─ 3300- Some Task.bes
```

/Fixlets/ — all .bes files (Fixlets, Tasks, Analyses). 
    Subdirectories auto-compile: Fixlets/Analyses/ → Analyses.fxf.
    Sub-Sub-Directories become Digests within an FXF, i.e. Fixlets.fxf contains digests for FixletType1 and FixletType2

/NonClientFiles/ — server-side assets, scripts, metadata (optional).

/OtherFiles/ — Files to be gathered by the client (optional).

/site.xml — global relevance rules applied to all Fixlets (optional).

/siteConfig.xml — customize default directory names (optional).

Filename convention: start with numeric ID, hyphen, space, filename --> '1- Task1.bes', '42- Fixlet Name.bes'.
The numeric ID portion of a filename becomes the Fixlet ID.
Filenames without a numeric ID, or duplicate numeric IDs, will not be gathered on the BigFix Server and will show a warning in the server logs.

# Add Repository Site using BigFix Console
In BigFix Console, go to Tools > Create Repository Site.

Enter the SSH URL: git@github.com:username/repo-name.git

Specify the branch to track (typically 'main' or 'master').

Optionally set a name and description for the site.

BigFix will verify SSH access using the keys in GitCredentials, then clone the repository.

Monitor the root server logs for messages like 'Repository site cloned successfully' or 'Repository site updated successfully'.
