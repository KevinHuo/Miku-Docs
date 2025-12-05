本文为您介绍了七牛云直播服务的所有API列表，建议您使用服务端 SDK 进行调用

## 空间管理

| 接口名称 | 描述 |
|:--------|:-----|
| [创建空间](/mikustream/api/13094/mikustream-live-bucket-create-api) | 创建一个新的空间（Bucket） |
| [删除空间](/mikustream/api/13095/mikustream-live-bucket-delete-api) | 删除指定的空间（Bucket） |
| [列举空间](/mikustream/api/13093/mikustream-live-bucket-list-api) | 获取当前账号下所有空间（Bucket）的列表信息 |
| [修改空间配置](/mikustream/api/13096/mikustream-live-bucket-config-update-api) | 修改指定空间（Bucket）的配置 |
| [获取空间配置](/mikustream/api/13097/mikustream-live-bucket-config-api) | 获取指定直播空间（Bucket）的配置信息 |


## 域名管理

| 接口名称 | 描述 |
|:--------|:-----|
| [绑定上行域名](/mikustream/api/13177/mikustream-live-bind-upstream-domain-api) | 为指定直播空间绑定上行域名（推流域名） |
| [解绑上行域名](/mikustream/api/13178/mikustream-live-unbind-upstream-domain-api) | 为指定直播空间解绑上行域名（推流域名） |
| [列举上行域名](/mikustream/api/13179/mikustream-live-list-upstream-domain-api) | 列举指定直播空间的所有上行域名（推流域名） |
| [修改上行域名配置](/mikustream/api/13181/mikustream-live-update-upstream-domain-config-api) | 修改指定直播空间中某个上行域名的配置 |
| [获取上行域名配置](/mikustream/api/13180/mikustream-live-get-upstream-domain-config-api) | 获取指定直播空间中某个上行域名的详细配置信息 |
| [绑定下行域名](/mikustream/api/13182/mikustream-live-bind-downstream-domain-api) | 为指定直播空间绑定下行域名（播放域名） |
| [解绑下行域名](/mikustream/api/13183/mikustream-live-unbind-downstream-domain-api) | 为指定直播空间解绑下行域名（播放域名） |
| [列举下行域名](/mikustream/api/13184/mikustream-live-list-downstream-domain-api) | 列举指定直播空间的所有下行域名（播放域名） |
| [修改下行域名配置](/mikustream/api/13186/mikustream-live-update-downstream-domain-config-api) | 修改指定直播空间中某个下行域名的配置 |
| [获取下行域名配置](/mikustream/api/13185/mikustream-live-get-downstream-domain-config-api) | 获取指定直播空间中某个下行域名的详细配置信息 |