```bash
echo "alias pwgen=\"LC_ALL=C tr -dc 'A-Za-z0-9@#%^&*()_+=-{}[]:;<>,.?/' \
    < /dev/urandom | head -c 20 | xargs echo\"" >> ~/.zshrc

# run with
pwgen
```