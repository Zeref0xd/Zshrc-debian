# ~/.zshrc (merged: zshrc1 features + zshrc2 prompt)

setopt autocd
setopt interactivecomments
setopt magicequalsubst
setopt nonomatch
setopt notify
setopt numericglobsort
setopt promptsubst

WORDCHARS=${WORDCHARS//\/}
PROMPT_EOL_MARK=""

bindkey -e
bindkey ' ' magic-space
bindkey '^U' backward-kill-line
bindkey '^[[3;5~' kill-word
bindkey '^[[3~' delete-char
bindkey '^[[1;5C' forward-word
bindkey '^[[1;5D' backward-word
bindkey '^[[5~' beginning-of-buffer-or-history
bindkey '^[[6~' end-of-buffer-or-history
bindkey '^[[H' beginning-of-line
bindkey '^[[F' end-of-line
bindkey '^[[Z' undo

autoload -Uz compinit
compinit -d ~/.cache/zcompdump
zstyle ':completion:*:*:*:*:*' menu select
zstyle ':completion:*' auto-description 'specify: %d'
zstyle ':completion:*' completer _expand _complete
zstyle ':completion:*' format 'Completing %d'
zstyle ':completion:*' group-name ''
zstyle ':completion:*' list-colors ''
zstyle ':completion:*' list-prompt %SAt %p: Hit TAB for more, or the character to insert%s
zstyle ':completion:*' matcher-list 'm:{a-zA-Z}={A-Za-z}'
zstyle ':completion:*' rehash true
zstyle ':completion:*' select-prompt %SScrolling active: current selection at %p%s
zstyle ':completion:*' use-compctl false
zstyle ':completion:*' verbose true
zstyle ':completion:*:kill:*' command 'ps -u $USER -o pid,%cpu,tty,cputime,cmd'

HISTFILE=~/.zsh_history
HISTSIZE=1000
SAVEHIST=2000
setopt hist_expire_dups_first
setopt hist_ignore_dups
setopt hist_ignore_space
setopt hist_verify

alias history="history 0"
TIMEFMT=$'\nreal\t%E\nuser\t%U\nsys\t%S\ncpu\t%P'

# -------------------------------
# Parrot-style prompt (VPN inline after [dir])
# -------------------------------
setopt prompt_subst

# async updater for VPN IP
update_vpn_ip() {
  local ip
  ip=$(ip -4 addr show tun0 2>/dev/null | grep -oP '(?<=inet\s)\d+(\.\d+){3}' | head -n1)
  if [[ -n $ip ]]; then
    VPN_IP="â”€[%F{3}$ip%F{1}]"
  else
    VPN_IP=""
  fi
}
# run once at start
update_vpn_ip
# refresh before each prompt
precmd_functions+=(update_vpn_ip)

if [[ $EUID == 0 ]]; then
  USER_COLOR="%B%F{1}"   # root = red
  PROMPT='%F{1}â”Œâ”€$([[ $? != 0 ]] && echo "[%F{7}âœ—%F{1}]â”€")[%f'$USER_COLOR'%n%B%F{3}@%F{14}%m%b%F{1}]â”€[%F{2}%~%F{1}]${VPN_IP}
â””â”€â”€â•¼ %B%F{3}# %b%f'
else
  USER_COLOR="%F{7}"     # user = white
  PROMPT='%F{1}â”Œâ”€$([[ $? != 0 ]] && echo "[%F{7}âœ—%F{1}]â”€")[%f'$USER_COLOR'%n%B%F{3}@%F{14}%m%b%F{1}]â”€[%F{2}%~%F{1}]${VPN_IP}
â””â”€â”€â•¼ %B%F{3}$ %b%f'
fi


# -------------------------------
# Syntax highlighting
# -------------------------------
if [ -f /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh ]; then
    . /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
    ZSH_HIGHLIGHT_HIGHLIGHTERS=(main brackets pattern)
    ZSH_HIGHLIGHT_STYLES[default]=none
    ZSH_HIGHLIGHT_STYLES[unknown-token]=underline
    ZSH_HIGHLIGHT_STYLES[reserved-word]=fg=cyan
    ZSH_HIGHLIGHT_STYLES[suffix-alias]=fg=green,underline
    ZSH_HIGHLIGHT_STYLES[global-alias]=fg=green
    ZSH_HIGHLIGHT_STYLES[precommand]=fg=white
    ZSH_HIGHLIGHT_STYLES[commandseparator]=fg=blue
    ZSH_HIGHLIGHT_STYLES[autodirectory]=fg=green,underline
    ZSH_HIGHLIGHT_STYLES[path]=none,bold
    ZSH_HIGHLIGHT_STYLES[globbing]=fg=blue
    ZSH_HIGHLIGHT_STYLES[history-expansion]=fg=blue
    ZSH_HIGHLIGHT_STYLES[single-hyphen-option]=fg=blue
    ZSH_HIGHLIGHT_STYLES[double-hyphen-option]=fg=yellow
    ZSH_HIGHLIGHT_STYLES[single-quoted-argument]=fg=yellow
    ZSH_HIGHLIGHT_STYLES[double-quoted-argument]=fg=yellow
    ZSH_HIGHLIGHT_STYLES[dollar-quoted-argument]=fg=yellow
    ZSH_HIGHLIGHT_STYLES[rc-quote]=fg=magenta
    ZSH_HIGHLIGHT_STYLES[redirection]=fg=blue
    ZSH_HIGHLIGHT_STYLES[comment]=fg=black
    ZSH_HIGHLIGHT_STYLES[arg0]=fg=cyan
    ZSH_HIGHLIGHT_STYLES[bracket-error]=fg=red
    ZSH_HIGHLIGHT_STYLES[bracket-level-1]=fg=blue
    ZSH_HIGHLIGHT_STYLES[bracket-level-2]=fg=green
    ZSH_HIGHLIGHT_STYLES[bracket-level-3]=fg=magenta
    ZSH_HIGHLIGHT_STYLES[bracket-level-4]=fg=yellow
    ZSH_HIGHLIGHT_STYLES[bracket-level-5]=fg=cyan
    ZSH_HIGHLIGHT_STYLES[cursor-matchingbracket]=standout
fi

# -------------------------------
# Autosuggestions
# -------------------------------
if [ -f /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh ]; then
    . /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
    ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=#999'
fi

# -------------------------------
# LS colors + aliases
# -------------------------------
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    export LS_COLORS="$LS_COLORS:ow=30;44:"
    alias ls='ls --color=auto'
    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
    alias diff='diff --color=auto'
    alias ip='ip --color=auto'
fi

alias ll='ls -lh'
alias la='ls -lha'
alias l='ls -CF'

# -------------------------------
# Command-not-found handler
# -------------------------------
if [ -f /etc/zsh_command_not_found ]; then
    . /etc/zsh_command_not_found
fi

alias ls="lsd"
alias ll="lsd -l"
alias la="lsd -a"

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

