## 拷贝 ssh keygen

```bash
xclip -sel clip < ~/.ssh/id_rsa.pub
```

## 部署指令

```bash
sh /var/server/idt-haw-deploy/front_nuxt.sh -p ${JOB_NAME} -w ${WORKSPACE}
```