# CS Learning Notes

这是我的计算机科学学习笔记站，使用 **Hugo + Blowfish** 构建，并通过 GitHub Pages 发布。

- 在线阅读：[boxijiekexun.github.io](https://boxijiekexun.github.io/)
- 个人主站：[boxijiekexun.com](https://boxijiekexun.com/)（独立网站，不是本仓库的部署地址）

这里会逐步记录课程学习、项目实践和技术排障；文章持续更新，并非完整的课程答案仓库。

## 本地预览

需要 Git 和 Hugo **Extended 0.167.0**（与当前部署工作流使用的版本一致）。

```bash
git clone --recurse-submodules https://github.com/boxijiekexun/boxijiekexun.github.io.git
cd boxijiekexun.github.io
hugo server -D
```

启动后访问 `http://localhost:1313/`。如果之前克隆时没有拉取主题子模块，可在仓库内运行 `git submodule update --init --recursive`。

## 发布

推送到 `main` 后，仓库的 [GitHub Actions 工作流](.github/workflows/hugo.yml) 会构建 Hugo 站点并部署到 GitHub Pages。
