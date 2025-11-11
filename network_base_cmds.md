```text
config t
hostname EDG
line console 0
logging synchronous
exec-timeout 0 0
no banner login
no banner exec
no banner incoming
end
copy running-config startup-config
```