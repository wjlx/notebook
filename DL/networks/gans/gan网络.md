# gan网络

&emsp;&emsp;GAN是由两部分主成：生成模型G（generative model）和判别模型D（discriminative model），生成模型捕捉样本数据的分布，判别模型是一个二分类器，判别输入是真实数据还是生成的样本。
![gan结构](../images/DL/gan结构.png)
&emsp;&emsp;X是真实数据，真实数据符合Pdata(x)分布。z是噪声数据，噪声数据符合Pz(z)分布，比如高斯分布或者均匀分布。然后从噪声z进行抽样，通过G之后生成数据x=G(z)。然后真实数据与生成数据一起送入分类器D，后面接一个sigmoid函数，输出判定类别。

我们先定义GAN的标记符号：

- Pdata(x) -> 真实数据的分布
- X -> pdata(x)的样本
- P(z) -> 生成器的分布
- Z -> p(z)的样本
- G(z) -> 生成网络
- D(x) -> 鉴别网络

$$
\begin{aligned}
\min _G \max _D V(D, G) = \Epsilon _{x \sim p_{data}(x)}[\log D(x)] +
\Epsilon _{x \sim p_z(x)} [\log(1 - D(G(z)))]
\end{aligned}
$$
