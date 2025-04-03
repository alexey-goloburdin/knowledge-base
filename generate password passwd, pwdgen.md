```bash
# Password generator
pwgen() {
  local len=${1:-20}
  LC_ALL=C tr -dc 'A-Za-z0-9@#%^&*()_+=-{}[]:;<>,.?/' < /dev/urandom | head -c "$len" | xargs echo
}
```

Этот alias уже добавлен в мои [dotfiles](https://github.com/alexey-goloburdin/dotfiles) в `.config/zsh/aliases.zsh`.

Использование:

```bash
# default 20 symbols
pwgen
Aq=?Wl+U%O10infrq)D

# 50 symbols
pwgen 50
Z_y3O%S9^u]C7GE{*2&5.xDNYO5/V@:u)(IBLvAi_}n:RhNc4

# 5 symbols
pwgen 5
abou,
```
