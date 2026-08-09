# Major Upgrade

Used to *start over* with a clean install, primarily required when GitLab
undergoes a major version update and/or new third-party dependencies are
introduced.

NOTE: The instructions below assume we are still resource-constrained and
      therefore need to move data and VMs around carefully.

Generally speaking, the process is:
  - Stand up a new GitLab instance at the new version in a Proxmox VM
    on the temporary Proxmox host
  - Migrate the data from the old GitLab to the new GitLab using direct
    transfer
  - Switch the `gitlab.westsidestreet.net` DNS A record from the old
    GitLab to the new GitLab
  - Verify everything is working
  - Shut down the old GitLab VM
  - Shut down the new GitLab and migrate the VM to the final destination

## Prerequisites

1.  On the *old* GitLab, export the Sealed Secret key(s) so that the new
    GitLab can decrypt existing keys.

    Ref: [How can I do a backup of my SealedSecrets?](https://github.com/bitnami/sealed-secrets#how-can-i-do-a-backup-of-my-sealedsecrets)

    ```bash
    kubectl get secret -n infrastructure -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml > sealed-secrets.keys.yaml
    ```

2.  On the *new* GitLab, import the Sealed Secret key(s) from above

    ```bash
    kubectl apply -f sealed-secrets.keys.yaml

    # Or, if running from a machine with access to multiple context:
    kubectl --context=gitlab-19 apply -f ~/backups/sealed-secrets/sealed-secrets-keys-gitlab01-2026-08-07.yaml
    ```

    NOTE: After applying the secrets above, restart the `sealed-secrets` Pod(s)

3.  On the *new* GitLab, use Ansible to add the repositories to ArgoCD (if new ArgoCD)

## Postrequisites

1.  Save the Sealed Secret key(s) from the *new* GitLab

2.  Update SSH keys in ArgoCD

3.  Update Helm repo project IDs in ArgoCD

## Random Notes

1.  Generating a password/token:

    ```bash
    openssl rand -base64 48

    tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 32; echo
    ```
