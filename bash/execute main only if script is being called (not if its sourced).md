```bash
# Only invoke url_encode if the script is being executed
# (rather than sourced).
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    url_encode $@
fi
```

