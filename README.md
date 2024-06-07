# 《数据可视化》小组作业 ———— 中国人口数据可视化

## 小组成员

张天行，李岳阳，张誉真

## 成果展示

本项目已通过 Github Pages 进行部署，URL 为[中国人口数据可视化](https://tristean9.github.io/PopulationVisual/)

## 项目结构

```plaintext
PopulationVisual
├─ 📁public
│  ├─ 📁data                    # 可视化使用到的数据
│  │  ├─ 📄不同年龄段人口占比.csv
│  │  ├─ 📄人口增长情况（2000-2023）.csv
│  │  ├─ 📄人口自然增长率数据.csv
│  │  ├─ 📄受教育程度.csv
│  │  ├─ 📄各因素对生育意愿的影响.csv
│  │  ├─ 📄各因素对生育意愿的影响.json
│  │  ├─ 📄男女性观念差别.csv
│  │  └─ 📄若干省生育率.csv
│  ├─ 📄china.json              # 中国地图数据
│  ├─ 📄introduction.png        # 项目介绍图片
│  └─ 📄favicon.ico             # 网站图标
├─ 📁src
│  ├─ 📁components
│  │  ├─ 📄Aging.vue            # 人口老龄化组件
│  │  ├─ 📄BirthNotDuty.vue     # 生育非义务化组件
│  │  ├─ 📄FemaleEdu.vue        # 女性受教育程度组件
│  │  ├─ 📄GenderDiff.vue       # 男女性观念差别组件
│  │  ├─ 📄Heatmap.vue          # 热力图组件
│  │  ├─ 📄Introduction.vue     # 项目介绍组件
│  │  ├─ 📄Overview.vue         # 数据概览组件
│  │  ├─ 📄Policy.vue           # 生育政策组件
│  │  ├─ 📄Reasons.vue          # 生育意愿影响因素组件
│  │  └─ 📄WordCloud.vue        # 词云图组件
│  ├─ 📄App.vue                 # Vue的根组件
│  ├─ 📄main.js                 # 项目的入口文件，用于初始化Vue实例等
│  └─ 📄style.css               # 项目的主要样式表
├─ 📄index.html                 # 项目的主HTML文件，通常作为单页面应用的入口
├─ 📄package-lock.json          # 自动生成的文件，描述项目依赖的具体版本，确保依赖一致性
├─ 📄package.json               # 描述项目的元数据和依赖关系的文件
├─ 📄README.md                  # 项目的README文件，用于提供项目说明、使用方法等信息
└─ 📄vite.config.js             # Vite的配置文件，用于自定义Vite的构建和开发行为
```

## 前提条件

在开始之前，请确保你的机器已经安装了以下软件：

- Node.js (可以从 [Node.js 官网](https://nodejs.org/) 下载安装)

```bash
# 检查 Node.js 安装
node --version
npm --version
```

### 前端 Vue 3 运行说明

1. 进入 `D3ProVisual` 目录：

    ```bash
    cd D3ProVisual
    ```

2. 安装依赖：

    ```bash
    npm install
    ```

3. 运行开发服务器：

    ```bash
    npm run dev
    ```

这将在本地启动前端开发服务器，默认在 [http://localhost:5173](http://localhost:5173)。即可在网页中查看项目报告
