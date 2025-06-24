## Part 1: Exporting Your GPG Secret Key (from Git Bash on Windows)

**Important:** Make sure you're in a **Git Bash terminal (MINGW64)** for these commands, as your GPG setup is specific to that environment.

1.  **List your secret GPG keys to find the exact Key ID.**
    Look for the **16-character string** after `0x` in the `sec` line. For example, if the output shows `sec   rsa4096/0xYOURKEYIDHERE`, you'll copy `YOURKEYIDHERE`.

    ```bash
    gpg --list-secret-keys --keyid-format LONG
    # Make sure the format is "LONG"
    ```

2.  **Export your secret key to a file.**
    **Replace `YOUR_16_CHAR_KEY_ID`** with the actual ID you copied from the previous step. A `pinentry` window will pop up asking for your GPG passphrase; enter it there.

    ```bash
    gpg --export-secret-keys --armor YOUR_16_CHAR_KEY_ID > github_private_key.asc
    ```

3.  **(Optional) Verify the exported file content.**
    This command will display the contents of the exported key file. It should start with `-----BEGIN PGP PRIVATE KEY BLOCK-----`.

    ```bash
    cat github_private_key.asc
    ```

4.  **(Manual Step) Securely transfer `github_private_key.asc` to your new machine/platform.**<br>
    **Do NOT** share this file publicly or send it unencrypted via email.<br>
    Use a secure method like encrypting the directory containing the key using 7z<br>
    or if you want to go extra, encrypt an entire USB with VeraCrypt that contains the key.

    * **Downloads:**<br>
        * [Download 7-Zip](https://www.7-zip.org/)<br>
        * [Download VeraCrypt](https://veracrypt.io/en/Downloads.html)<br>

    * **Tutorials:**<br>
        * 7-Zip Tutorial:<br>
          **IMPORTANT, MUST WATCH IF YOU USE 7-Zip!!!**<br>
          [7-Zip Tutorial Video](https://youtu.be/icG__m92tRY?si=ndUaURf4bpWlk80j&t=86) (Contains info about security issue)

        * VeraCrypt Tutorial:<br>
          [VeraCrypt Docs Tutorial](https://veracrypt.io/en/Beginner's%20Tutorial.html) (Recomended over videos)

---

## Part 2: Importing GPG Key and Setting Up Automatic Signing (on New Platform)

**Important:** Make sure to set up your git!<br>
* [Official Guide](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)

(We will continue under the assumption that you used the standard windows installer)<br>
The first thing you should do when you install Git is to set your user name and email address.
```bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

### Now to the Importing

1.  **(Manual Step) Place `github_private_key.asc` on the new machine.**
    Navigate your terminal or command prompt to the directory where you placed this file.

2.  **Import the secret key into GPG's keyring on the new platform.**
    You will be prompted for your GPG passphrase by `pinentry`.

    ```bash
    gpg --import github_private_key.asc
    ```

3.  **(Optional) Verify the key has been imported successfully on the new platform.**

    ```bash
    gpg --list-secret-keys --keyid-format LONG
    ```

4.  **(Optional) Set the key's trust level (usually 'ultimate').**
    **Replace `YOUR_16_CHAR_KEY_ID`** with your key's ID (which you found in the previous step). In the prompt that appears, type `5` for 'I trust ultimately', then `y` to confirm, and finally `quit`.

    ```bash
    gpg --edit-key YOUR_16_CHAR_KEY_ID
    # (Inside the gpg prompt, type 'trust' then '5' then 'y' then 'quit')
    ```

5.  **Configure Git to use this GPG key for signing.**
    **Replace `YOUR_16_CHAR_KEY_ID`** with your key's ID.

    ```bash
    git config --global user.signingkey YOUR_16_CHAR_KEY_ID
    ```

6.  **Enable automatic GPG signing for all your Git commits.**

    ```bash
    git config --global commit.gpgsign true
    ```

7.  **(Optional but Recommended) Configure GPG to use `gpg-agent` for caching passphrases.**
    This prevents you from being prompted for your passphrase on every commit for a while.
    Open or create the GPG configuration file, which is often located at `~/.gnupg/gpg-agent.conf`. On Windows, this is typically `C:\Users\YourUser\.gnupg\gpg-agent.conf`. Add the following lines to the file:

    ```
    default-cache-ttl 600
    max-cache-ttl 7200
    ```
    (The numbers are in seconds: `600s = 10 minutes`, `7200s = 2 hours`). You might need to **restart your terminal or kill existing `gpg-agent` processes** for this to take effect. To kill `gpg-agent` on Windows, open Task Manager, find `gpg-agent.exe`, and select 'End task'. It will restart automatically.

8.  **Test a signed commit.**
    Make a dummy commit in any Git repository to confirm signing works. You should be prompted for your GPG passphrase once, but then not again for the configured caching period. Push the commit to GitHub or GitLab, and you should see the "Verified" badge.
    To check locally:

    ```bash
    git commit -m "Test commit with GPG signing"
    git log --show-signature -1
    ```

9.  **(Manual Step) DELETE `github_private_key.asc` from your system**
    Do this **only after** you have confirmed the key is successfully imported and working on the new platform. This file contains your private key and should not be kept as a permanent copy in an easily accessible location.

---