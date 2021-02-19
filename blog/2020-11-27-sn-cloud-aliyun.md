---
title: Pulsar 云服务，落地阿里云
categories: "engineering"
image: "/blog/media/aliyun/cover-free-cloud.png"
topBackgroundImage: "/blog/media/aliyun/top-free-cloud.png"
description: "作为一种全面托管、可扩展的消息传递和事件流服务，StreamNative Cloud 为希望在云平台构建和启动事件流应用程序的公司提供了一套完整的解决方案。在阿里云平台部署 StreamNative Cloud，也有助于中国的公司采用 Pulsar。"
summary: "今天，我们很高兴地宣布在阿里云平台部署 StreamNative Cloud。StreamNative Cloud 支持简单、快速、可靠、且经济高效地在云平台上运行 Pulsar。通过将 StreamNative Cloud 部署在阿里云平台，StreamNative 可以为中国客户提供专业的 Pulsar 运维和管理支持，促进中blog国各大公司采用 Pulsar。"
displayDate: "November 27, 2020"
tags: "StreamNative Cloud, 阿里云"
authorList: ["yang","carolyn-king"]
reviewer: ["sijie"]
id: "2020-11-27-sn-cloud-aliyun"
---

今年八月份，Apache 顶级项目 Pulsar 背后的开源流数据公司 StreamNative 宣布推出基于 Apache Pulsar 的云端服务产品——StreamNative Cloud。现在，StreamNative 又决定将其 StreamNative Cloud 部署在阿里云平台。

阿里云是中国最大的云提供商。将 StreamNative Cloud 部署在阿里云平台，有助于 StreamNative 服务新的客户群。 StreamNative 首席执行官郭斯杰表示：“通过将 StreamNative Cloud 部署在阿里云平台，StreamNative 可以为中国客户提供专业的 Pulsar 运维和管理支持，促进中国各大公司采用 Pulsar“。

# Apache Pulsar 的成长之路

2018 年 9 月，Pulsar 成为 Apache 的顶级项目。此后，Pulsar 在全球范围内迅猛发展。越来越多的公司采用 Pulsar 构建创新的应用程序并借助实时流解决方案改进其现有系统。根据 [Docker Hub](https://hub.docker.com/) 的统计，Apache Pulsar 的下载量超过 1000 万。在 Github，Pulsar 项目已收获大约 7,000 个点赞，吸引 330 多位贡献者参与项目开发和维护。在过去的两年，Pulsar 项目贡献者的数量增长了十倍。

![](/blog/media/aliyun/pulsar-growth.png)

在北美、欧洲和亚洲等地区，越来越多的公司正在使用 Pulsar。

# Pulsar 在中国的发展应用

自项目启动以来，Pulsar 在亚洲地区得到了广泛采用。实际上，四年前，Yahoo! JAPAN 已经开始采用 Pulsar。最初，Yahoo! JAPAN 采用 Pulsar 构建新的统一消息平台。现在，Yahoo! JAPAN 使用 Pulsar 作为其消息传递主干，每天处理数千亿条消息。

智联招聘，中国领先的在线招聘平台，也是 Pulsar 的早期用户之一。Pulsar 支持强大的功能，具有极高的灵活性。自 2018 年以来，智联招聘一直使用 Pulsar，每天处理数千亿条消息。

越来越多的中国公司使用 Pulsar，并参与Pulsar 项目的开发与维护。例如，腾讯利用 Pulsar 构建其交易计费系统 Midas。Midas 规模庞大，每天处理 100 多亿笔金融交易、10 TB 以上的数据。这也直接证明 Pulsar 处理关键任务应用程序的能力。

此外，中国的公司也高度参与 Pulsar 社区。今年初，中国移动与 StreamNative 合作开发并推出 AMQP on Pulsar（AoP）。AoP 支持使用 RabbitMQ（或其他 AMQP 消息 broker ）的公司在不修改代码的情况下将现有应用程序和服务迁移到 Pulsar。Pulsar 生态系统还新增了许多其他功能。这使得 Pulsar 日益强大，可以满足更多的实时数据需求。同时，这也说明 Pulsar 的活跃社区也可以推动 Pulsar 项目的持续发展。

现在，腾讯、中国移动、Yahoo! JAPAN 和智联招聘等一百多家公司都在采用 Pulsar。Pulsar 的应用已遍布电信、互联网、零售、电子商务、金融、物联网等多个行业。

StreamNative 将于 11月 28 日至 29 日主办首届 Pulsar 亚洲峰会。届时，来自 Splunk、Yahoo! JAPAN、TIBCO、中国移动、腾讯、达达-京东到家，金山软件、涂鸦智能和 PingCAP 的技术负责人、开源开发人员、软件工程师和软件架构师齐聚一堂，进行 30 多场会议。会议议程涉及 Pulsar 用例、Pulsar 生态系统、Pulsar 运营和 Pulsar 技术深入探讨。单击[此处](https://hopin.to/events/pulsar-summit-asia-2020)，注册参加 Pulsar 亚洲峰会。

# 为什么选择阿里云

在阿里云平台部署 StreamNative Cloud 是我们的一项战略性决策。阿里云是中国最大的公有云提供商，拥有中国最大的云网络，能够为中国快速增长的 Pulsar 社区提供服务。
作为一种全面托管、可扩展的消息传递和事件流服务，StreamNative Cloud 为希望在云平台构建和启动事件流应用程序的公司提供了一套完整的解决方案。在阿里云平台部署 StreamNative Cloud，也有助于中国的公司采用 Pulsar。

# 关于 StreamNative Cloud

StreamNative Cloud 由 Apache Pulsar 和 Apache BookKeeper 的初创开发人员进行构建和运营，为企业提供了可扩展、灵活、安全的消息传递和事件流平台。翼支付（Bestpay） 的首席数据科学家谢巍盛（Vincent）对 StreamNative 的产品印象深刻。他表示：“StreamNative Cloud 助力我们快速启动灵活、安全、可扩展的事件流服务。StreamNative Cloud 简单易用，极大地提高了我们团队的工作效率。”

目前 StreamNative Cloud 已部署在阿里云平台。您可以单击[此处](https://console.cloud.streamnative.cn/?defaultMethod=signup)，试用 StreamNative Cloud 服务。

想要随时掌握 Pulsar 的研发进展、用户案例和热点话题吗？快来关注 Apache Pulsar 和 StreamNative 微信公众号，我们第一时间在这里分享与 Pulsar 有关的一切。