# 就看天气

----

### 前言
最初上线是在2015年10月，是自己第一个较为成熟的应用，开发完之后刚好答了知乎这篇[如何自学Android编程][1]

但是因为代码确实写的很烂，所以决定全部重构代码全新风格的展示就看天气Ver2.0.

当然自己也在学习之中，如果发现有任何问题和建议，随时欢迎Email或者开Issues


- **开源不易，希望能给个Star鼓励**
- 项目地址：https://github.com/xcc3641/SeeWeather
- 项目主页发布issue: https://github.com/xcc3641/SeeWeather/issues
- 本项目为开源项目,技术交流可以通过邮箱联系: Hugo3641@gmail.com

### 简介
就看天气——是一款遵循**Material Design**风格的只看天气的APP。无流氓权限，无自启，xxx，用最少的权限做最优的体验。
- 卡片展现（当前天气情况，未来几小时天气情况，生活建议，一周七天概况）
- 补全城市（第一版本因为自己偷懒所以城市有缺陷对不起各位）
- 自动定位
- 缓存数据，减少网络请求，保证离线查看
- 内置两套图标（设置里更改）
- 彩蛋（自动夜间状态）


----

权限说明

```
	<!--用于进行网络定位-->
	<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
	<!--用于访问GPS定位-->
	<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
	<!--获取运营商信息，用于支持提供运营商信息相关的接口-->
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
	<!--用于访问wifi网络信息，wifi信息会用于进行网络定位-->
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
	<!--这个权限用于获取wifi的获取权限，wifi信息会用来进行网络定位-->
	<uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
	<!--用于访问网络，网络定位需要上网-->
	<uses-permission android:name="android.permission.INTERNET"/>
	<!--用于读取手机当前的状态-->
	<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
	<!--写入扩展存储，向扩展卡写入数据，用于写入缓存定位数据-->
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
	<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

```

### 版本更新&&下载地址
Fir.im: http://fir.im/seeWeather

酷安市场：http://www.coolapk.com/apk/com.xiecc.seeWeather

豌豆荚：http://www.wandoujia.com/apps/com.xiecc.seeWeather

魅族应用中心： http://developer.meizu.com/console/apps/detail/6530883

v2.1.6
- 更新 RxUtil
- 优化 网络和缓存逻辑
- 更多的封装

v2.1.3
- 修复 城市列表卡顿
- 更新 lambda
- 新增 bugly
- 修复 高德地图未注销监听器导致的内存泄露

v2.1
- 修复 定位逻辑
- 优化 SP的统一化
- 优化 ERRO界面
- 更新 新的天气 ICON
- 更新 通知栏提醒（后台可能需要白名单，不然无法自动更新）
- 设置 自动更新频率（0 为不自动更新）


v2.0
- 重构代码，全新UI，升级体验
- 就看天气——是一款遵循**Material Design**风格的只看天气的APP。无流氓权限，无自启，xxx，用最少的权限做最优的体验。
- 卡片展现（当前天气情况，未来几小时天气情况，生活建议，一周七天概况）
- 彩蛋（自动夜间状态）
- 补全城市（第一版本因为自己偷懒所以城市有缺陷对不起各位）
- 缓存数据，减少网络请求，保证离线查看
- 内置两套图标（设置里更改）

v1.0
- 就看天气V1.0
- @图片和信息来源于网络，侵权删




### TODO
这学期有点忙，需要花时间巩固基础，准备面试，但是自己还是会抽空尽快做出这些功能的，谢谢大家理解和支持

- [ ] 桌面小部件
- [x] 通知栏提醒
- [x] 更好，更多的天气ICONS
- [ ] 管理城市（多城市选择）
- [x] 自动定位
- [ ] 自由定制的Item界面



----

### 项目
#### 公开 API

天气数据来源于：和风天气

城市信息来源于：CSDN

地理定位服务： 高德地图

#### 开源技术
1. [Rxjava][2]
2. [RxAndroid][3]
3. [Retrofit][4]
4. [GLide][5]
5. [ASimpleCache][6]

#### 代码

##### 网络
就看天气的网络部分的支持是用`RxJava+RxAndroid+Retrofit+Gson`再加上`ACache`缓存

网络部分：
配合 RetrofitSingleton 中封装的方法：
```java
public Observable<Weather> fetchWeather(String city) {
		return apiService.mWeatherAPI(city, C.KEY)
				.filter(weatherAPI -> weatherAPI.mHeWeatherDataService30s.get(0).status.equals("ok"))
				.map(weatherAPI -> weatherAPI.mHeWeatherDataService30s.get(0))
				.compose(RxUtils.rxSchedulerHelper());
}
```

RxUtils 工具类中封装了线程调度：

```java
public static <T> Observable.Transformer<T, T> rxSchedulerHelper() {
		return tObservable -> tObservable.subscribeOn(Schedulers.io())
				.unsubscribeOn(AndroidSchedulers.mainThread())
				.observeOn(AndroidSchedulers.mainThread());
}
```

感受下实际操作

```java
    private Observable<Weather> fetchDataByNetWork() {
        String cityName = Util.replaceCity(mSetting.getCityName());
        return RetrofitSingleton.getInstance()
            .fetchWeather(cityName)
            .onErrorReturn(throwable -> {
                PLog.e(throwable.getMessage());
                return null;
            });
    }
```

一定要写 onErrorRrturn ，不然是不会执行到缓存的流。

因为和风天气 API 有些城市/省份的末尾的特殊符号需要过滤掉，比如 `省|市|自治区|特别行政区|地区|盟`,所以在我的方法类中写了一个安全的替换方法。

缓存部分：
```java
private Observable<Weather> fetchDataByCache() {
		return Observable.defer(() -> {
						Weather weather = (Weather) aCache.getAsObject(C.WEATHER_CACHE);
						return Observable.just(weather);
				}
		).compose(RxUtils.rxSchedulerHelper());
}
```

使用 concat 连接：
优先网络数据

```java
private void load() {
        Observable.concat(fetchDataByNetWork(), fetchDataByCache())
            .first(weather -> weather != null)
            .doOnError(throwable -> {
                mErroImageView.setVisibility(View.VISIBLE);
                mRecyclerView.setVisibility(View.GONE);
                Snackbar.make(fab, "网络不好,~( ´•︵•` )~", Snackbar.LENGTH_INDEFINITE).setAction("重试", v -> {
                    load();
                }).show();
            })
            .doOnNext(weather -> {
                mErroImageView.setVisibility(View.GONE);
                mRecyclerView.setVisibility(View.VISIBLE);
            })
            .doOnTerminate(() -> {
                mRefreshLayout.setRefreshing(false);
                mProgressBar.setVisibility(View.GONE);
            })
            .subscribe(observer);
    }
```

##### RecyclerView 展示

就像洪洋说的一样

> 整体上看RecyclerView架构，提供了一种插拔式的体验，高度的解耦，异常的灵活，通过设置它提供的不同LayoutManager，ItemDecoration , ItemAnimator实现令人瞠目的效果。

该项目中用到 RecyclerView 中的用法是根据 itemType 展示不同的布局，这就是主页 UI 核心的代码了。

```
@Override public int getItemViewType(int position) {
	    if (position == TYPE_ONE) {
	    //标识
	        ...
	    }
	    return super.getItemViewType(position);
	}

@Override public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
	    if (viewType == TYPE_ONE) {
	    //绑定
	        ...
	        }
	    }
   }

@Override public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
	    //更新布局
	    ....

}

```



----


### 截图
![][image-1]
![][image-2]
![][image-3]

### 感谢
感谢开源，学习到了前辈们优秀的代码
- [@张鸿洋][7]
- [@扔物线][8]
- [@drakeet][9]
- [@代码家][10]
- [@程序亦非猿][11]
- [@小鄧子][12]
- [@Jude95][13]
- [@泡在网上编代码][14]

特别感谢**简书猿圈**

### 关于作者

![][image-4]

简书：http://www.jianshu.com/users/3372b4a3b9e5/latest\_articles

知乎：https://www.zhihu.com/people/xcc3641.github.io

微博：http://weibo.com/xcc3641

个人博客： http://IMXIE.CC


### 请我喝杯咖啡

----

![][image-5]
----

### LICENSE

Copyright 2016 HugoXie  Licensed under the Apache License, Version 2.0 (the \"License\")

you may not use this file except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an \"AS IS\" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

图片来源于网络，版权属于原作者。






[1]:	https://www.zhihu.com/question/26417244/answer/70193822
[2]:	https://github.com/ReactiveX/RxJava
[3]:	https://github.com/ReactiveX/RxAndroid
[4]:	https://github.com/square/retrofit
[5]:	https://github.com/bumptech/glide
[6]:	https://github.com/yangfuhai/ASimpleCache
[7]:	https://github.com/hongyangAndroid
[8]:	https://github.com/rengwuxian
[9]:	https://github.com/drakeet
[10]:	https://github.com/daimajia
[11]:	https://github.com/AlanCheen
[12]:	https://github.com/SmartDengg
[13]:	https://github.com/Jude95
[14]:	http://weibo.com/u/2711441293?topnav=1&wvr=6&topsug=1&is_all=1

[image-1]:	http://xcc3641.qiniudn.com/app-%E5%B0%B1%E7%9C%8B%E5%A4%A9%E6%B0%94-%E9%97%AA%E5%B1%8F.png
[image-2]:	http://xcc3641.qiniudn.com/app-%E5%B0%B1%E7%9C%8B%E5%A4%A9%E6%B0%94-%E7%99%BD%E5%A4%A9%E6%A8%A1%E5%BC%8F.png
[image-3]:	http://xcc3641.qiniudn.com/app-%E5%B0%B1%E7%9C%8B%E5%A4%A9%E6%B0%94-%E5%A4%9C%E6%99%9A%E6%A8%A1%E5%BC%8F.png
[image-4]:	http://xcc3641.qiniudn.com/app-%E5%A4%B4%E5%83%8F-1.jpeg
[image-5]:	http://xcc3641.qiniudn.com/app-%E6%94%AF%E4%BB%98%E5%AE%9D.jpg
