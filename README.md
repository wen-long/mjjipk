### mjjipk
一个适合 mjj 的 IP 数据库, 注重易读和在命令行中使用, 包含位置和 AS 信息, 以 mmdb 方式提供
#### 数据源
混合了 qqwry, ip2location, ipinfo 的信息,规则如下

0. 知名的国内段, 采用 qqwry 的信息, 如 114.114.112.0/21, CN2 59.43
1. qqwry 和 ipinfo 都定位为 CN 的段, 采用 qqwry 的信息(更适合国内用户)
2. ip2location 和 ipinfo 国家定位相同时, 采用 ip2location 的信息(ipinfo 只到国家一级)
3. ip2location 和 ipinfo 国家定位不一致时, 采用 ipinfo 信息
4. 如果可用, 则使用 ipinfo 的 AS 信息

### 数据格式
只提供 mmdb, 可以使用工具/库解析, 也支持 wireshark 直接使用

```
mmdbinspect --db mjjipk.mmdb 211.1.1.1 223.5.5.5
[
    {
        "Database": "mjjipk.mmdb",
        "Records": [
            {
                "Network": "211.1.1.1/19",
                "Record": {
                    "city": {
                        "names": {
                            "en": "AS2522 nic.ad.jp"
                        }
                    },
                    "country": {
                        "iso_code": "JP",
                        "names": {
                            "en": "JP 東京 Tokyo"
                        }
                    }
                }
            }
        ],
        "Lookup": "211.1.1.1"
    },
    {
        "Database": "mjjipk.mmdb",
        "Records": [
            {
                "Network": "223.5.5.5/32",
                "Record": {
                    "city": {
                        "names": {
                            "en": "阿里巴巴anycast公共DNS"
                        }
                    },
                    "country": {
                        "iso_code": "CN",
                        "names": {
                            "en": "浙江–杭州"
                        }
                    }
                }
            }
        ],
        "Lookup": "223.5.5.5"
    }
]
```
### 如何使用
这个库最初被设计用来配合 https://github.com/zu1k/nali 使用, 下载到 nali 的数据库目录后, 只需要在 config.yaml 中添加项目即可
```
    - name: mjjipk
      format: mmdb
      file: mjjipk.mmdb
      languages:
        - en_US
```
在 wireshark 中使用时, 在高级中搜索 mmdb, 配置 `nameres.maxmind_db_paths` 为 mjjipk.mmdb 所在目录的路径  
使用 `ip.geoip.city` 等[相关过滤器](https://www.wireshark.org/docs/dfref/i/ip.html)
<img width="788" alt="image" src="https://github.com/user-attachments/assets/1d3335b2-0183-4b1f-af0d-b3f70d0328f5">


### 更新和维护
目前依赖 GitHub Actions 每周更新一次

### 这是如何做到的

对三个数据库排序遍历拆解合并, 生成最细 IP 段的最大“事实表”后, 重新遍历并应用业务逻辑  
中间数据示例:
```

{1298125056 1298126847 {意大利  CZ88.NET} {IT Italy Piemonte Settimo Torinese} {IT Italy EU Europe AS43185 RESO' SRL reso.it}}
{1298126848 1298127359 {沙特阿拉伯  CZ88.NET} {SA Saudi Arabia Makkah al Mukarramah Jeddah} {      }}
{1298127360 1298127871 { } {   } {SA Saudi Arabia AS Asia   }}
{1298127872 1298128127 {伊朗  CZ88.NET} {IR Iran (Islamic Republic of) Tehran Tehran} {      }}
{1298128128 1298128895 {沙特阿拉伯  CZ88.NET} {SA Saudi Arabia Makkah al Mukarramah Jeddah} {      }}
{1298128896 1298130943 {荷兰  CZ88.NET} {NL Netherlands (Kingdom of the) Zuid-Holland Schiedam} {NL Netherlands EU Europe AS62370 Snel.com B.V. snel.com}}
{1298130944 1298131455 {保加利亚  CZ88.NET} {BG Bulgaria Sofia (stolitsa) Sofia} {BG Bulgaria EU Europe AS25332 Linkos Ltd. linkos.bg}}

{3658238327 3658238327 {中国–河北–石家庄 联通/深蓝网吧} {CN China Hebei Shijiazhuang} {CN China AS Asia AS4837 CHINA UNICOM China169 Backbone chinaunicom.cn}}
{3658238328 3658238328 {中国–河北–石家庄 联通} {CN China Hebei Shijiazhuang} {CN China AS Asia AS4837 CHINA UNICOM China169 Backbone chinaunicom.cn}}
{3658238329 3658238329 {中国–河北–石家庄 联通/百盛网络广场} {CN China Hebei Shijiazhuang} {CN China AS Asia AS4837 CHINA UNICOM China169 Backbone chinaunicom.cn}}
{3658238330 3658238381 {中国–河北–石家庄 联通} {CN China Hebei Shijiazhuang} {CN China AS Asia AS4837 CHINA UNICOM China169 Backbone chinaunicom.cn}}

{1746728757 1746728764 {美国  CZ88.NET} {LU Luxembourg Luxembourg Luxembourg} {ZA South Africa AF Africa AS13335 Cloudflare, Inc. cloudflare.com}}
{1746728765 1746728770 {美国  CZ88.NET} {LU Luxembourg Luxembourg Luxembourg} {SV El Salvador NA North America AS13335 Cloudflare, Inc. cloudflare.com}}
{1746728771 1746728778 {美国  CZ88.NET} {LU Luxembourg Luxembourg Luxembourg} {HK Hong Kong AS Asia AS13335 Cloudflare, Inc. cloudflare.com}}
{1746728779 1746728788 {美国  CZ88.NET} {LU Luxembourg Luxembourg Luxembourg} {IN India AS Asia AS13335 Cloudflare, Inc. cloudflare.com}}
{1746728789 1746728796 {美国  CZ88.NET} {LU Luxembourg Luxembourg Luxembourg} {KZ Kazakhstan AS Asia AS13335 Cloudflare, Inc. cloudflare.com}}
{1746728797 1746728801 {美国  CZ88.NET} {LU Luxembourg Luxembourg Luxembourg} {MH Marshall Islands OC Oceania AS13335 Cloudflare, Inc. cloudflare.com}}
{1746728802 1746728810 {美国  CZ88.NET} {LU Luxembourg Luxembourg Luxembourg} {US United States NA North America AS13335 Cloudflare, Inc. cloudflare.com}}
{1746728811 1746728820 {美国  CZ88.NET} {LU Luxembourg Luxembourg Luxembourg} {IN India AS Asia AS13335 Cloudflare, Inc. cloudflare.com}}
```
