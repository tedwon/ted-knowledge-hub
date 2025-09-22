
Yes, you absolutely can log in from the CLI with a password.1

You can provide the username and password directly using the **`-u`** and **`-p`** flags with the `oc login` command.

---

## **Logging in with Username and Password**

1. **Get Credentials:** First, run `crc console` to get the server URL and the password for the `kubeadmin` user.2
    
    Bash
    
    ```
    crc console
    ```
    
2. **Run the Login Command:** Use the information from the previous step to build your command.
    
    Bash
    
    ```
    oc login <your-server-url> -u kubeadmin -p <your-password>
    ```
    
    For example, it will look something like this:
    
    Bash
    
    ```
    oc login https://api.crc.testing:6443 -u kubeadmin -p <paste-your-password-here>
    ```
    

This will log you in directly without any interactive prompts.

**Note:** Be aware that putting your password directly in the command line can save it to your shell's history file, which can be a security risk on shared systems.