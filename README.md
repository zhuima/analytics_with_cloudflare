# Web Analytics with Cloudflare + Huno + D1 database

[Cloudflare D1 Doc](https://developers.cloudflare.com/d1/get-started/)

## Setup

### Log in

```bash
npx wrangler login
```

### Create Database

```
npx wrangler d1 create <DATABASE_NAME>

---------
✅ Successfully created DB '<DATABASE_NAME>'

[[d1_databases]]
binding = "DB" # available in your Worker on env.DB
database_name = "<DATABASE_NAME>"
database_id = "<unique-ID-for-your-database>"

```

### Bind Worker to D1 database

in **wrangler.toml**

```
[[d1_databases]]
binding = "DB" # available in your Worker on env.DB
database_name = "<DATABASE_NAME>"
database_id = "<unique-ID-for-your-database>"
```

### init database

```
npm run initSql
```

### Deploy

```
$ npm run deploy


> analytics_with_cloudflare@0.0.0 deploy
> wrangler deploy

Proxy environment variables detected. We'll use your proxy for fetch requests.
 ⛅️ wrangler 3.18.0
-------------------
Your worker has access to the following bindings:
- D1 Databases:
  - DB: web_analytics (<unique-ID-for-your-database>)
Total Upload: 50.28 KiB / gzip: 12.23 KiB
Uploaded analytics_with_cloudflare (1.29 sec)
Published analytics_with_cloudflare (4.03 sec)
  https://analytics_with_cloudflare.xxxxx.workers.dev
Current Deployment ID: xxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Use

inclout js and add `<span id='page_pv'>?</span> <span id='page_uv'>?</span>`

```
<script src="/front/dist/index.min.js" data-base-url="https://analytics_with_cloudflare.xxxxx.workers.dev"></script>
<script src="/front/dist/index.min.js" data-base-url="diy Url"></script>
<script src="/front/dist/index.min.js" data-base-url="diy Url" data-page-pv-id="page_pv" data-page-uv-id="page_uv"></script>
```

- data-base-url: Default value is `https://webviso.yestool.org`
- data-page-pv-id: Default value is `page_pv`
- data-page-uv-id: Default value is `page_uv`

## Example

- 由于cloudflare对应的域名在国内不能正常访问，所以建议绑定自定义域名
- 传递参数有5个，如下

### curl

```
curl  -X POST \
  'https://analytics.chuhai.tools/api/visit' \
  --header 'Content-Type: application/json' \
  --data-raw '{
  "url": "https://chuhai.tools/tools/about-privacy-1ts-fun",
  "hostname": "https://chuhai.tools",
  "referrer": "",
  "pv": true,
  "uv": true
}'
```

### Next.js

```
- app/api/pageview/route.ts
import { respErr } from "@/app/utils/resp";
import { getPageView } from "@/app/services/pageview";

export async function POST(req: Request) {
  try {
    const params = await req.json();

    console.log("api router params", params);

    const response = await getPageView(
      params.url,
      params.hostname,
      params.referrer
    );
    return response;
  } catch (e) {
    console.log("get page view failed: ", e);
    const errinfo = "get page view failed: " + e;
    return respErr(errinfo);
  }
}


- app/service/pageview.ts
export function getPageView(url: string, hostname: string, referrer: string) {
  return fetch("https://analytics.chuhai.tools/api/visit", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      url,
      hostname,
      referrer,
      pv: true,
      uv: true,
    }),
  });
}

- app/utils/resp.ts
export function respErr(message: string) {
  return respJson(-1, message);
}

- app/utils/resp.ts
export function respJson(code: number, message: string, data?: any) {
  let json = {
    code: code,
    message: message,
    data: data,
  };
  if (data) {
    json["data"] = data;
  }

  return Response.json(json);
}


```
