As define in `package.json`

```json
"scripts": {
    "build": "hexo generate",
    "clean": "hexo clean",
    "deploy": "hexo deploy",
    "server": "hexo server",
    "CGS": "hexo clean && hexo generate && hexo server",
    "CGSD": "hexo clean && hexo generate && hexo server && hexo deploy"
  }
```

- generate & server

```bash
npm run CGS
```

- deploy

```bash
npm run CGSD
```