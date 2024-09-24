# 域名购买到博客配置的过程记录


这篇记录详细描述了我从购买域名到配置博客的全过程, 包括选择域名, 注册步骤, DNS 设置及博客平台配置等关键环节.
旨在帮助读者了解每一步的具体操作, 为他们的博客创建提供参考.
这里感谢下赛博佛祖: `Cloudfalre`

<!--more-->

## 选择方案

因为在国内域名要备案, 秉承着 `INTP` 的懒人思想, 能避免麻烦就避免选择了在国外注册域名, 当然也想着为以后能做点海外的产品或者网站先铺铺路.
通过 `Google` 和询问了群里的大佬, 一般都是在哪里购买域名, 推荐是

- [Cloudflare](https://www.cloudflare.com/)
- [namecheap](https://www.namecheap.com/)

对比了下 2 个价格, namecheap 会便宜些, 所以就通过 namecheap 注册了

## 选择域名

我一直以来用的 AKA 或者网名都是: L_MaFia , 根据 lmafia 搜索,哪个便宜买哪个, 比如 `.info`, `.xyz`, `.life` 啥的.
`.info` 一年只需要 $2.4 左右, 注意购买的时候最好关闭自动续费,一般续费价格就不划算了,我也当做是练手,可能不一定会使用超过一年时间.

## 支付方式

`namecheap` 支持使用 `Visa`, `PayPal` 一些常用的国外支付方式, 这也应该难倒了不少网友了,如果我没有 `Visa` 卡, 全币卡咋办？
其实,`PayPal` 是可以关联咱们的银联的储蓄卡的,这次购买一年的域名的花费总购不在 **27RMB**.

我选择了一张基本不使用的建设银行的储蓄卡,但是在付费扣款时一直出现失败

询问了群里的大佬,应该是有踩坑的经验: 国内一些银联卡可能会无法支付,他是用招商银行的卡支付的

我记得我本科的时候,使用 `PayPal` 购买过 `DigitalOcean` 的服务器时用过工行卡,想着还是听大佬的先试试看,为了保险起见的话,打不了支付成功后再解绑即可.

实测下来:

- 建设银行-无法支付
- 招商银行-成功支付

## 让域名指向博客

我的博客是用 `Hugo` 搭建的纯静态的网站,白嫖了 `Vercel` 和 `GithPage`, 体验下来 `Vercel` 的访问速度会好些,当然我后续我可能会尝试白嫖 `Cloudflare Pages`

### Cloudfalre 添加域名

{{< image src="https://image-s3.lmafia.info/data/2024/09/de60df2613b1133f7364f412109858ab.png" caption="cf-domain-add" width="960" >}}

### Cloudflare 的免费 SSL

想用 `HTTPS` 就避免不了需要使用 `SSL`,不仅因为安全考虑,还有如果需要一些域名解析指向应用服务平台,平台也会一定的安全要求 ,`Cloudflare` 免费提供了 `SSL` 够我们开发者的博客内容使用了

{{< image src="https://image-s3.lmafia.info/data/2024/09/96c64f4a47229bc5b7bfb92f89c51031.png" caption="cf-domain-ssl" width="960" >}}

一定要选择 `Strict` , 不然 `Vercel` 是不会给是指向过去的

### Cloudfalre 提供 DNS 服务

{{< image src="https://image-s3.lmafia.info/data/2024/09/601c619ea66d25bec14f5bb9eed3b574.png" caption="cf-domain-dns" width="960" >}}

### Namecheap 配置 DNS

`Namecheap` 提供也 `DNS` 了,但是因为我们需要 **白嫖** `SSL`
所以需要修改下 `DNS` 服务列表
`Namecheap` 提供的教程:

- [How to set up DNS records for your domain in a Cloudflare account](https://www.namecheap.com/support/knowledgebase/article.aspx/9607/2210/how-to-set-up-dns-records-for-your-domain-in-a-cloudflare-account/)

看其中的第三和第四步,大概也就了解了

配置好这个之后,可以说我们的域名就可以交给 `Cloudfalre` 来 「管理」 了

### Cloudfalre 添加 DNS Record

这一步就是给用户访问域名的时候, 需要浏览到我们的具体网页地址上
这里我添加了一个 `CNAME` 规则

- `CNAME`: 域名解析为另外一个域名

{{< image src="https://image-s3.lmafia.info/data/2024/09/96c64f4a47229bc5b7bfb92f89c51031.png" caption="cf-domain-dns-cname" width="960" >}}

当我输入浏览器输入: `blog.lmafia.info` 时, 会解析到目标地址: `Vercel` 的博客域名

### `Vercel` 域名添加配置

最后一步就是在 `Vercel` 给我们博客项目配置域名
我们一直配置好 Cloudfalre 到 `Vercel` 的单向的配置了
我估计是 `Vercel` 因为安全原因,还需要配置这个域名并会自动验证
验证可用会有 2 个 ✅ 出现

{{< image src="https://image-s3.lmafia.info/data/2024/09/c897ac0953232354a4cc6862a95228c7.png" caption="vercel-domain-add" width="960" >}}

## 完成配置

接下来, 让我们通过域名访问下该站点吧: https://blog.lmafia.info

成功访问!

恭喜,我们已经完成了给我们的博客配置一个属于自己的域名啦!

