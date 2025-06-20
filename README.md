# 必应壁纸

[[🇩🇪 Deutsch 德语](README_de.md)] [[🇨🇦 🇬🇧 🇮🇳 🇺🇸 English 英语](README_en.md)] [[🇪🇸 Español 西班牙语](README_es.md)] [[🇨🇦 🇫🇷 Français 法语](README_fr.md)] [[🇮🇹 Italiano 意大利语](README_it.md)] [[🇯🇵 日本語 日语](README_ja.md)] [[🇧🇷 Português 葡萄牙语](README_pt.md)]

## 区域列表

| 区域代码 | 时区                                                              | 备注                             |
| :------: | :---------------------------------------------------------------- | :------------------------------- |
| `de-DE`  | 冬令时：`UTC+1`，夏令时：`UTC+2`，[来源](https://time.is/Germany) |                                  |
| `en-CA`  | 冬令时：`UTC-5`，夏令时：`UTC-4`，[来源](https://time.is/Canada)  |                                  |
| `en-GB`  | 冬令时：`UTC+0`，夏令时：`UTC+1`，[来源](https://time.is/England) |                                  |
| `en-IN`  | `UTC+5:30`，[来源](https://time.is/India)                         | `hi-IN` 与其几乎完全一样，也舍弃 |
| `en-US`  | 冬令时：`UTC-8`，夏令时：`UTC-7`，[来源](https://time.is/Redmond) |                                  |
| `es-ES`  | 冬令时：`UTC+1`，夏令时：`UTC+2`，[来源](https://time.is/Spain)   |                                  |
| `fr-CA`  | 冬令时：`UTC-5`，夏令时：`UTC-4`，[来源](https://time.is/Canada)  |                                  |
| `fr-FR`  | 冬令时：`UTC+1`，夏令时：`UTC+2`，[来源](https://time.is/France)  |                                  |
| `it-IT`  | 冬令时：`UTC+1`，夏令时：`UTC+2`，[来源](https://time.is/Italy)   |                                  |
| `ja-JP`  | `UTC+9`，[来源](https://time.is/Japan)                            |                                  |
| `pt-BR`  | `UTC-3`，[来源](https://time.is/Brazil)                           |                                  |
| `zh-CN`  | `UTC+8`，[来源](https://time.is/China)                            |                                  |

详细可见 [regional_list.csv](regional_list.csv)

## 数据源

[https://services.bingapis.com/ge-apps/api/v2/bwc/hpimages?mkt=](https://services.bingapis.com/ge-apps/api/v2/bwc/hpimages?mkt=)

返回的数据结构用 [zod](https://zod.dev/) 表示如下。

```javascript
z.object({
  images: z.array(
    z.object({
      startdate: z.string().regex(/^\d{8}$/),
      fullstartdate: z.string().regex(/^\d{12}$/),
      enddate: z.string().regex(/^\d{8}$/),
      urlbase: z
        .string()
        .regex(
          /^https:\/\/www\.bing\.com\/th\?id=OHR\.([A-Za-z0-9]+)_(DE-DE|EN-CA|EN-GB|EN-IN|EN-US|ES-ES|FR-CA|FR-FR|IT-IT|JA-JP|PT-BR|ZH-CN)(\d+)_UHD\.jpg$/
        ), // 已知 en-GB 区域存在不符合该模式的数据
      copyrighttext: z.string().startsWith("© "),
      copyrightlink: z
        .string()
        .regex(
          /^https:\/\/www\.bing\.com\/search\?q=([^&]+)&form=BGALM(?:&filters=HpDate:"(\d{8}_\d{4})")$/
        ), // 已知 en-GB 区域存在不符合该模式的数据
      title: z.string(),
      description: z.string(),
      headline: z.string(),
      quiz: z
        .string()
        .regex(
          /^https:\/\/www\.bing\.com\/search\?q=Bing\+homepage\+quiz&filters=WQOskey:"HPQuiz_(\d{8})_([^"]+)"&FORM=BGAQ$/
        ), // 已知 en-GB 区域存在不符合该模式的数据
      mapLink: z.object({
        Url: z
          .string()
          .startsWith(
            "https://platform.bing.com/geo/REST/v1/Imagery/Map/RoadVibrant/"
          ),
        Link: z.string().startsWith("https://www.bing.com/maps?"),
      }),
      quizId: z.null(), // 未找到除 null 以外的值
      fullDateString: z.string(), // 当地语言的当地日期
      theme: z.tuple([z.literal("bing")]),
      travelUrl: z.string().startsWith("https://www.bing.com/"),
      visualSearchUrl: z.string(), // 一般为空字符串
      sourceType: z.literal("BingImageOfTheDay"),
      showVisualSearch: z.boolean(), // 一般为 false
      CTAData: z.null(), // 未找到除 null 以外的值
      detectedRegion: z.string(), // 取决于发送请求的地区
      enableBingImageCreator: z.boolean(), // 一般为 false
      topRightCTAData: z.looseObject({}), // 不重要的数据
      imageHotspots: z.null(), // 未找到除 null 以外的值
      AnimatedWP: z.null(), // 未找到除 null 以外的值
    })
  ),
  tooltips: z.looseObject({}), // 不重要的数据
  imageCount: z.literal(8),
});
```

## 字段列表

| 字段名            | 来源                                                    | 备注                 |
| :---------------- | :------------------------------------------------------ | :------------------- |
| `date`            | `images[].fullstartdate` 附加时区偏移（见上）得到的日期 | 只有日期部分         |
| `full_start_time` | `images[].fullstartdate`                                | UTC 时间，精确到小时 |
| `image_url`       | `images[].urlbase`                                      | 见注释 1             |
| `copyright`       | `images[].copyrighttext`                                |                      |
| `search_url`      | `images[].copyrightlink`                                |                      |
| `title`           | `images[].title`                                        |                      |
| `description`     | `images[].description`                                  |                      |
| `headline`        | `images[].headline`                                     |                      |
| `quiz_url`        | `images[].quiz`                                         |                      |
| `map_image`       | `images[].mapLink.Url`                                  | 见注释 2             |
| `map_url`         | `images[].mapLink.Link`                                 | 见注释 2             |

1. `image_url` 默认返回横屏 4K 图片（3840x2160），可以通过附加 `&w=<宽>&h=<高>&rs=1&c=4` 参数来裁剪图片，但要注意不能超过图片的最大尺寸。
2. `map_image` 和 `map_url` 目前只在以下区域的部分图片上提供：`de-DE`、`en-CA`、`en-GB`、`en-IN`、`en-US`、`fr-CA`、`fr-FR`、`it-IT`、`ja-JP`。
