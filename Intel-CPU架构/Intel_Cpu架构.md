# [Intel核显的一些解析](https://zhuanlan.zhihu.com/p/102049728)
# [Products formerly Tiger Lake](https://ark.intel.com/content/www/us/en/ark/products/codename/88759/products-formerly-tiger-lake.html#@Mobile)
# [Intel® Processor Graphics Documentation](https://www.intel.com/content/www/us/en/developer/articles/guide/intel-graphics-developers-guides.html)
# [Intel’s Next Generation Integrated Graphics Architecture](https://www.intel.com/Assets/PDF/whitepaper/313343.pdf)

# PC开发概览
## 规划，确定产品规格
芯片选项、ID、重量、散热规格（CPU TDP、GPU TDP）、电池大小等。
## 硬件开发、结构开发
硬件电路板设计、电源top图、效率
## BIOS、EC固件开发
1个项目平均需要投入3人-5人的bios工程师+1个EC工程师用于bios开发、bug修正。对比手机在uboot的工作量增加很多。
## 驱动适配
器件厂商不提供源代码，发布二进制驱动安装包给ODM、OEM厂商，OEM厂商需要自行验证驱动的功能、兼容性。驱动正式发布时需要通过WHQL认证。
## Image制作适配
OEM厂商会开发工具、管家类软件，制作安装Image后预装。
## 产测工具软件开发
OEM、ODM厂商在工厂用于测试、老化、维测，确保PC功能张昌
## 总结
新增了BIOS、EC固件的开发，无驱动源代码、无OS源代码、缺少商用交付后的大点、维测等信息。
