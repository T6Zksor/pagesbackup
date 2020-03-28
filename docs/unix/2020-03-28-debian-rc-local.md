# Debian 9 使用 rc.local
2020-03-28

Debian 9 默认没有开启 `rc-local`，开启步骤

1. 手工添加 `/etc/rc.local` 文件，内容为

    ```
    #!/bin/sh -e
    #
    # rc.local
    #
    # This script is executed at the end of each multiuser runlevel.
    # Make sure that the script will "exit 0" on success or any other
    # value on error.
    #
    # In order to enable or disable this script just change the execution
    # bits.
    #
    # By default this script does nothing.

    exit 0
    ```

2. `chmod +x /etc/rc.local`

3. `systemctl start rc-local`
