# Default Configuration Files

All the config files here are meant to be examples and should list either all, as many as needed, or required configuration parameters.

## Config Catalogue

|  File Name   | Description                                                                      | Source                                                                                                                                  |
| :----------: | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
|  wsl.config  | Configure settings per-distribution for Linux distros running on WSL 1 or WSL 2. | [Advanced settings configuration in WSL](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#wslconf)                               |
|  .wslconfig  | Configure settings globally across all installed distributions running on WSL 2  | [Advanced settings configuration in WSL](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#wslconfig)                             |
|   .bashrc    | Default Ubuntu .bashrc file                                                      | Pulled from [Official Ubuntu (20.04) Docker image](https://hub.docker.com/_/ubuntu) on 2022-04-18                                       |
|    .zshrc    | Default .zshrc file                                                              | [Oh My Zsh zsh template](https://github.com/ohmyzsh/ohmyzsh/blob/eb00b95d26e8f264af80f508d50ac32e50619027/templates/zshrc.zsh-template) |
| defaults.vim | Default `vim` configuration file                                                 | [Vim repository](https://github.com/vim/vim/blob/53e8f3ffdf80dbd24a60adb51f8f21982fd41c57/runtime/defaults.vim)                         |

## Sources Lists

Collection of apt sources list, in case you wrecked yours like I wrecked mine trying to use stuff like `apt-mirror-updater`...