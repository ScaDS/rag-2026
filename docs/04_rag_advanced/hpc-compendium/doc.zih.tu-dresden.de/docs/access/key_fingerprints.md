---
search:
  boost: 10.0
---

# Key Fingerprints

!!! hint

    The key fingerprints of login and dataport nodes can occasionally change. This page holds
    up-to-date fingerprints.

Each cluster can be accessed via so-called **login nodes** using specific hostnames. All login nodes
of a cluster share common SSH key fingerprints that are unique. When connecting to one of our HPC
systems via the corresponding login nodes, please make sure that the provided fingerprint matches
one of the table. This is the only way to ensure you are connecting to the correct server from the
very beginning. If the **fingerprint differs**, please contact the
[HPC support team](../support/support.md).

In case an additional login node of the same cluster is used, the key needs to be checked and
approved again.

??? example "Connecting with SSH to Barnard"

    ```console
    marie@local$ ssh login1.barnard.hpc.tu-dresden.de
    The authenticity of host 'login1.barnard.hpc.tu-dresden.de (172.24.95.28)' can't be established.
    ECDSA key fingerprint is SHA256:8Coljw7yoVH6HA8u+K3makRK9HfOSfe+BG8W/CUEPp0.
    Are you sure you want to continue connecting (yes/no)?
    ```

    In this case, the fingerprint matches the one given in the table. Thus, you can proceed by
    typing 'yes'.

??? info "Verify key fingerprints without login"

    To verify the fingerprint without login, use `ssh-keyscan` and `ssh-keygen`:

    ```console
    marie@local$ ssh-keyscan login1.barnard.hpc.tu-dresden.de 2>/dev/null | ssh-keygen -l -f -
    3072 SHA256:lVQOvnci07jkxmFnX58pQf3cD7lz1mf4K4b9jZrAlVU login1.barnard.hpc.tu-dresden.de (RSA)
    256 SHA256:Xan2MYazewT0V5agNazaQfWzLKBD3P48zRwR6reoXhI login1.barnard.hpc.tu-dresden.de (ECDSA)
    256 SHA256:Gn4n5IX9eEvkpOGrtZzs9T9yAfJUB200bgRchchiKAQ login1.barnard.hpc.tu-dresden.de (ED25519)
    ```

## Change of Key Fingerprints

The message "REMOTE HOST IDENTIFICATION HAS CHANGED" means that your SSH client has detected a
change in the server's host key. This is a security warning designed to prevent man-in-the-middle
attacks.

When you first connect to an SSH server (e.g. our login and dataport nodes), its host key is stored
in the `~/.ssh/known_hosts` file on your local system. On subsequent connections, the SSH client
checks whether the server's host key matches the stored one. If the key has changed, SSH will refuse
to connect and display a corresponding warning.

There are two possible reasons for this warning:

1. Legitimate changes: The HPC admin team intentionally changed the SSH host key. If so, you will
   find the updated key fingerprint for the node on this page and you need to manually remove the
   old key from the file `~/.ssh/known_hosts`.
1. Security concerns: There is a potential man-in-the-middle attack, or a compromised node that
   has been replaced with a malicious system. If you do not find the offered key fingerprint on this
   page, do not proceed and send a
   [ticket to the HPC admin team](mailto:hpc-support@tu-dresden.de?subject=HPC:%20Changed%20key%20fingerprint&body=Dear%20HPC-support%20team,%0A%0Aplease%20check%20the%20key%20fingerprint%20which%20is%20not%20documented%5B1%5D:%0A%0A-%20node:%0A-%20offered%20key%20fingerprint:%0A%0A%5B1%5D%20https%3A//compendium.hpc.tu-dresden.de/access/key_fingerprints/%0A%0ABest%20regards)

## Barnard

The cluster [`Barnard`](../jobs_and_resources/hardware_overview.md#barnard) can be accessed via the
four login nodes `login[1-4].barnard.hpc.tu-dresden.de`. (Please choose one concrete login node when
connecting, see example below.)

--8<--
docs/access/key_fingerprints_barnard_table.txt
{: summary="List of valid fingerprints for Barnard login[1-4] nodes"}
--8<--

## Romeo

The cluster [`Romeo`](../jobs_and_resources/hardware_overview.md#romeo) can be accessed via the two
login nodes `login[1-2].romeo.hpc.tu-dresden.de`. (Please choose one concrete login node when
connecting, see example below.)

--8<--
docs/access/key_fingerprints_romeo_table.txt
{: summary="List of valid fingerprints for Romeo login[1-2] node"}
--8<--

## Capella

The cluster [`Capella`](../jobs_and_resources/hardware_overview.md#capella) can be accessed via the
two login nodes `login[1-2].capella.hpc.tu-dresden.de`. (Please choose one concrete login node when
connecting, see example below.)

--8<--
docs/access/key_fingerprints_capella_table.txt
{: summary="List of valid fingerprints for Capella login[1-2] node"}
--8<--

## Alpha Centauri

The cluster [`Alpha Centauri`](../jobs_and_resources/hardware_overview.md#alpha-centauri) can be
accessed via the two login nodes `login[1-2].alpha.hpc.tu-dresden.de`. (Please choose one concrete
login node when connecting, see example below.)

--8<--
docs/access/key_fingerprints_alpha_table.txt
{: summary="List of valid fingerprints for Alpha Centauri login[1-2] node"}
--8<--

## Julia

The cluster [`Julia`](../jobs_and_resources/hardware_overview.md#julia) can be accessed via
`julia.hpc.tu-dresden.de`.  (Note, there is no separate login node.)

--8<--
docs/access/key_fingerprints_julia_table.txt
{: summary="List of valid fingerprints for Julia login node"}
--8<--

## Dataport Nodes

When you transfer files using the [dataport nodes](../data_transfer/dataport_nodes.md), please make
sure that the fingerprint shown matches one of the table.

--8<--
docs/access/key_fingerprints_dataport_table.txt
{: summary="List of valid fingerprints for dataport[1-2].hpc.tu-dresden.de"}
--8<--
