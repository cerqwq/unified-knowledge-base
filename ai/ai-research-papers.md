# AI 重要论文集

## 一、基础论文

### 1.1 机器学习基础

**1. "A Few Useful Things to Know about Machine Learning" (2012)**
- 作者：Pedro Domingos
- 内容：机器学习实践指南
- 要点：特征工程、过拟合、评估
- 影响：入门必读

**2. "The Elements of Statistical Learning" (2001)**
- 作者：Hastie, Tibshirani, Friedman
- 内容：统计学习理论
- 要点：回归、分类、聚类
- 影响：经典教材

**3. "Pattern Recognition and Machine Learning" (2006)**
- 作者：Christopher Bishop
- 内容：模式识别
- 要点：概率模型、贝叶斯方法
- 影响：理论基础

### 1.2 深度学习基础

**4. "Deep Learning" (2015)**
- 作者：LeCun, Bengio, Hinton
- 内容：深度学习综述
- 要点：CNN、RNN、优化
- 影响：深度学习宣言

**5. "ImageNet Classification with Deep Convolutional Neural Networks" (2012)**
- 作者：Krizhevsky, Sutskever, Hinton
- 内容：AlexNet
- 要点：CNN、ReLU、Dropout
- 影响：深度学习革命

**6. "Dropout: A Simple Way to Prevent Neural Networks from Overfitting" (2014)**
- 作者：Srivastava et al.
- 内容：Dropout 技术
- 要点：随机丢弃神经元
- 影响：正则化标准

**7. "Batch Normalization: Accelerating Deep Network Training" (2015)**
- 作者：Ioffe, Szegedy
- 内容：批归一化
- 要点：加速训练、稳定梯度
- 影响：训练技术标准

---

## 二、计算机视觉

### 2.1 图像分类

**8. "Very Deep Convolutional Networks for Large-Scale Image Recognition" (2014)**
- 作者：Simonyan, Zisserman
- 内容：VGGNet
- 要点：小卷积核、深度网络
- 影响：网络设计

**9. "Going Deeper with Convolutions" (2014)**
- 作者：Szegedy et al.
- 内容：GoogLeNet/Inception
- 要点：Inception 模块
- 影响：网络设计

**10. "Deep Residual Learning for Image Recognition" (2015)**
- 作者：He et al.
- 内容：ResNet
- 要点：残差连接
- 影响：深层网络

### 2.2 目标检测

**11. "You Only Look Once: Unified, Real-Time Object Detection" (2015)**
- 作者：Redmon et al.
- 内容：YOLO
- 要点：单次检测
- 影响：实时检测

**12. "Faster R-CNN: Towards Real-Time Object Detection" (2015)**
- 作者：Ren et al.
- 内容：Faster R-CNN
- 要点：区域提议网络
- 影响：检测框架

**13. "SSD: Single Shot MultiBox Detector" (2015)**
- 作者：Liu et al.
- 内容：SSD
- 要点：多尺度检测
- 影响：检测框架

### 2.3 图像生成

**14. "Generative Adversarial Networks" (2014)**
- 作者：Goodfellow et al.
- 内容：GAN
- 要点：生成器、判别器
- 影响：生成模型

**15. "Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks" (2015)**
- 作者：Radford, Metz, Chintala
- 内容：DCGAN
- 要点：CNN 生成
- 影响：图像生成

**16. "Image-to-Image Translation with Conditional Adversarial Networks" (2016)**
- 作者：Isola et al.
- 内容：Pix2Pix
- 要点：条件 GAN
- 影响：图像翻译

**17. "Denoising Diffusion Probabilistic Models" (2020)**
- 作者：Ho, Jain, Abbeel
- 内容：DDPM
- 要点：扩散模型
- 影响：图像生成

### 2.4 视觉 Transformer

**18. "An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale" (2020)**
- 作者：Dosovitskiy et al.
- 内容：ViT
- 要点：视觉 Transformer
- 影响：CV 新范式

**19. "Swin Transformer: Hierarchical Vision Transformer using Shifted Windows" (2021)**
- 作者：Liu et al.
- 内容：Swin Transformer
- 要点：层次化、窗口注意力
- 影响：视觉 Transformer

---

## 三、自然语言处理

### 3.1 词嵌入

**20. "Efficient Estimation of Word Representations in Vector Space" (2013)**
- 作者：Mikolov et al.
- 内容：Word2Vec
- 要点：CBOW、Skip-gram
- 影响：词嵌入

**21. "GloVe: Global Vectors for Word Representation" (2014)**
- 作者：Pennington, Socher, Manning
- 内容：GloVe
- 要点：全局统计
- 影响：词嵌入

### 3.2 序列模型

**22. "Sequence to Sequence Learning with Neural Networks" (2014)**
- 作者：Sutskever, Vinyals, Le
- 内容：Seq2Seq
- 要点：编码器-解码器
- 影响：序列建模

**23. "Neural Machine Translation by Jointly Learning to Align and Translate" (2014)**
- 作者：Bahdanau, Cho, Bengio
- 内容：注意力机制
- 要点：对齐、翻译
- 影响：注意力机制

### 3.3 预训练模型

**24. "Attention Is All You Need" (2017)**
- 作者：Vaswani et al.
- 内容：Transformer
- 要点：自注意力、多头注意力
- 影响：NLP 新范式

**25. "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding" (2018)**
- 作者：Devlin et al.
- 内容：BERT
- 要点：双向预训练
- 影响：NLP 革命

**26. "Language Models are Unsupervised Multitask Learners" (2019)**
- 作者：Radford et al.
- 内容：GPT-2
- 要点：零样本学习
- 影响：大语言模型

**27. "Language Models are Few-Shot Learners" (2020)**
- 作者：Brown et al.
- 内容：GPT-3
- 要点：少样本学习
- 影响：大语言模型

**28. "Scaling Laws for Neural Language Models" (2020)**
- 作者：Kaplan et al.
- 内容：缩放定律
- 要点：规模与性能
- 影响：大模型设计

---

## 四、强化学习

### 4.1 基础算法

**29. "Playing Atari with Deep Reinforcement Learning" (2013)**
- 作者：Mnih et al.
- 内容：DQN
- 要点：深度强化学习
- 影响：游戏 AI

**30. "Human-level control through deep reinforcement learning" (2015)**
- 作者：Mnih et al.
- 内容：DQN 改进
- 要点：经验回放、目标网络
- 影响：Nature 发表

**31. "Proximal Policy Optimization Algorithms" (2017)**
- 作者：Schulman et al.
- 内容：PPO
- 要点：裁剪目标函数
- 影响：策略优化

### 4.2 游戏 AI

**32. "Mastering the game of Go with deep neural networks and tree search" (2016)**
- 作者：Silver et al.
- 内容：AlphaGo
- 要点：蒙特卡洛树搜索
- 影响：里程碑

**33. "Mastering the game of Go without human knowledge" (2017)**
- 作者：Silver et al.
- 内容：AlphaGo Zero
- 要点：自我博弈
- 影响：无监督学习

**34. "Mastering Atari, Go, chess and shogi by planning with a learned model" (2020)**
- 作者：Schrittwieser et al.
- 内容：MuZero
- 要点：模型预测
- 影响：通用游戏 AI

---

## 五、生成模型

### 5.1 GAN 改进

**35. "Progressive Growing of GANs" (2017)**
- 作者：Karras et al.
- 内容：ProGAN
- 要点：渐进式训练
- 影响：高质量生成

**36. "A Style-Based Generator Architecture for Generative Adversarial Networks" (2018)**
- 作者：Karras et al.
- 内容：StyleGAN
- 要点：风格控制
- 影响：人脸生成

**37. "Analyzing and Improving the Image Quality of StyleGAN" (2019)**
- 作者：Karras et al.
- 内容：StyleGAN2
- 要点：质量改进
- 影响：图像生成

### 5.2 扩散模型

**38. "Denoising Diffusion Probabilistic Models" (2020)**
- 作者：Ho, Jain, Abbeel
- 内容：DDPM
- 要点：扩散过程
- 影响：扩散模型

**39. "High-Resolution Image Synthesis with Latent Diffusion Models" (2021)**
- 作者：Rombach et al.
- 内容：Latent Diffusion
- 要点：潜空间扩散
- 影响：Stable Diffusion

**40. "Classifier-Free Diffusion Guidance" (2022)**
- 作者：Ho, Salimans
- 内容：无分类器引导
- 要点：条件生成
- 影响：图像生成

---

## 六、多模态

### 6.1 视觉-语言

**41. "Learning Transferable Visual Models From Natural Language Supervision" (2021)**
- 作者：Radford et al.
- 内容：CLIP
- 要点：对比学习
- 影响：多模态

**42. "Zero-Shot Text-to-Image Generation" (2021)**
- 作者：Ramesh et al.
- 内容：DALL-E
- 要点：文本到图像
- 影响：生成模型

**43. "Hierarchical Text-Conditional Image Generation with CLIP Latents" (2022)**
- 作者：Ramesh et al.
- 内容：DALL-E 2
- 要点：层次生成
- 影响：图像生成

### 6.2 多模态大模型

**44. "GPT-4 Technical Report" (2023)**
- 作者：OpenAI
- 内容：GPT-4
- 要点：多模态、推理
- 影响：大模型

**45. "Gemini: A Family of Highly Capable Multimodal Models" (2023)**
- 作者：Google
- 内容：Gemini
- 要点：多模态原生
- 影响：大模型

---

## 七、AI Agent

### 7.1 Agent 框架

**46. "Toolformer: Language Models Can Teach Themselves to Use Tools" (2023)**
- 作者：Schick et al.
- 内容：Toolformer
- 要点：工具使用
- 影响：Agent

**47. "ReAct: Synergizing Reasoning and Acting in Language Models" (2022)**
- 作者：Yao et al.
- 内容：ReAct
- 要点：推理+行动
- 影响：Agent

**48. "Reflexion: Language Agents with Verbal Reinforcement Learning" (2023)**
- 作者：Shinn et al.
- 内容：Reflexion
- 要点：自我反思
- 影响：Agent

### 7.2 多 Agent

**49. "AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation" (2023)**
- 作者：Wu et al.
- 内容：AutoGen
- 要点：多 Agent 对话
- 影响：多 Agent

**50. "CrewAI: A Framework for Orchestrating Role-Playing AI Agents" (2024)**
- 作者：Moura
- 内容：CrewAI
- 要点：角色扮演
- 影响：多 Agent

---

## 八、AI 安全

### 8.1 对抗攻击

**51. "Explaining and Harnessing Adversarial Examples" (2014)**
- 作者：Goodfellow, Shlens, Szegedy
- 内容：FGSM
- 要点：对抗样本
- 影响：安全研究

**52. "Towards Deep Learning Models Resistant to Adversarial Attacks" (2017)**
- 作者：Madry et al.
- 内容：对抗训练
- 要点：鲁棒性
- 影响：防御方法

### 8.2 隐私保护

**53. "Deep Learning with Differential Privacy" (2016)**
- 作者：Abadi et al.
- 内容：差分隐私深度学习
- 要点：隐私保护
- 影响：隐私研究

**54. "Communication-Efficient Learning of Deep Networks from Decentralized Data" (2016)**
- 作者：McMahan et al.
- 内容：联邦学习
- 要点：分布式学习
- 影响：隐私保护

### 8.3 对齐

**55. "Training language models to follow instructions with human feedback" (2022)**
- 作者：Ouyang et al.
- 内容：InstructGPT
- 要点：RLHF
- 影响：对齐技术

**56. "Constitutional AI: Harmlessness from AI Feedback" (2022)**
- 作者：Bai et al.
- 内容：Constitutional AI
- 要点：自我批评
- 影响：对齐技术

---

## 九、应用论文

### 9.1 医疗 AI

**57. "Dermatologist-level classification of skin cancer with deep neural networks" (2017)**
- 作者：Esteva et al.
- 内容：皮肤癌诊断
- 要点：医学影像
- 影响：医疗 AI

**58. "Clinically applicable deep learning for diagnosis and referral in retinal disease" (2018)**
- 作者：De Fauw et al.
- 内容：视网膜疾病诊断
- 要点：医学影像
- 影响：医疗 AI

### 9.2 蛋白质

**59. "Highly accurate protein structure prediction with AlphaFold" (2021)**
- 作者：Jumper et al.
- 内容：AlphaFold
- 要点：蛋白质折叠
- 影响：科学发现

**60. "Highly accurate protein structure prediction for the human proteome" (2021)**
- 作者：Varadi et al.
- 内容：AlphaFold 数据库
- 要点：蛋白质组
- 影响：生物医学

---

## 十、综述论文

### 10.1 深度学习综述

**61. "Deep learning in neural networks: An overview" (2015)**
- 作者：Schmidhuber
- 内容：深度学习综述
- 要点：历史、方法、应用
- 影响：全面综述

**62. "A survey on transfer learning" (2010)**
- 作者：Pan, Yang
- 内容：迁移学习综述
- 要点：方法、应用
- 影响：迁移学习

### 10.2 NLP 综述

**63. "A Survey of Large Language Models" (2023)**
- 作者：Zhao et al.
- 内容：大语言模型综述
- 要点：架构、训练、应用
- 影响：LLM 综述

**64. "A Survey on Vision Transformer" (2020)**
- 作者：Han et al.
- 内容：视觉 Transformer 综述
- 要点：方法、应用
- 影响：ViT 综述

### 10.3 Agent 综述

**65. "A Survey on Large Language Model based Autonomous Agents" (2023)**
- 作者：Wang et al.
- 内容：LLM Agent 综述
- 要点：架构、应用
- 影响：Agent 综述

**66. "The Rise and Potential of Large Language Model Based Agents: A Survey" (2023)**
- 作者：Xi et al.
- 内容：LLM Agent 综述
- 要点：框架、应用
- 影响：Agent 综述

---

*本论文集收录 AI 领域重要论文，持续更新中。*
