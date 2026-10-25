# UniFi Network

## Connect AP

If an AP does not show up automatically, you can SSH into it while still in
factory settings.

```
ssh ubnt@{the-AP's-ip}
```

Password is `ubnt`.

Then run `set-inform http://{the-server's-ip}:8080/inform` and it should show up
in the server's web interface.
