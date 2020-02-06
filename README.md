# Prerequisites

- `Node.js`（版本需不低于`8.10`，建议使用`10.0`及以上版本）
- `Git`
- `Pandoc`（在Ubuntu上使用apt下载`Pandoc`可能会因版本过低出现问题，建议直接去![官方下载](https://github.com/jgm/pandoc/releases/latest)）

# Install

```bash
sudo npm install -g hexo-cli
git clone git@github.com:panjingyu/hexo-blog-source.git
cd hexo-blog-source
git submodule update --init --recursive
npm install
```
