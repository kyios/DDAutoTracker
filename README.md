# 无埋点编码规范
无埋点方案基于窗口回调(Window.Callback)机制。BaseActivity中集成了自动打点相关逻辑。但由于dialog和activity实现机制不一样。为了dialog同样能够集成自动打点功能，解决方案如下：
任何Dialog,需要继承DDDialog。
任何PopupWindow需要继承DDPopupWindow。

## 支持自动打点的控件
A.列表类
DDListView 普通listview，使用者可以在布局中直接使用ListView标签,系统会自动将ListView 的实现替换成DDListView，DDListView 内部会设置Item View的OnClickListener,并将点击事件委托给onItemClickListener。
DDExpandableListView 二级列表控件，使用者可以在布局中直接使用ExpandableListView标签,系统会自动将ExpandableListView 的实现替换成DDExpandableListView。DDExpandableListView 内部会设置Item View的OnClickListener,并将点击事件委托给onGroup/ChildClickListener

B.网格类
DDGridView 普通GridView,使用者可以在布局中直接使用GridView 标签，系统会自动将GridView 的实现替换成DDGridView。DDGridView 内部会设置Item View的OnClickListener,并将点击事件委托给onItemClickListener。

C.ViewPager
任何ViewPager 实现，只要相应的PagerAdapter 子类实现DDPagerAdapter,该ViewPager 控件即支持自动打点。

D.RecyclerView
任何RecyclerView实现，只要相应的RecyclerView.Adapter子类实现DDRecyclerAdapter,该RecyclerView 支持自动打点

以上控件中嵌套其它可自动打点控件,比如DDListView,自动打点实现基于最内层的自动打点控件。
请在开发过程中，尽量使用支持自动打点的控件，如果使用的控件暂不支持自动打点，请联系作者(刘硕)。

## 不支持自动打点控件的解决方案
对于自定义ViewGroup 等不支持自动打点的情况，打点框架提供绑定数据的操作。使用者可以在BaseActivity子类中通过方法configLayout将view和数据绑定，绑定之后该View下的任意子View的点击事件的埋点数据都会指向这条数据。对于在非Activity中实现打点数据绑定，可以调用AutoTrackHelper 中相应的方法,调用之前最好调用canxxxxx()进行绑定操作是否允许的检查。

对于某些需要手动发送埋点，但是同样需要走埋点框架的情况，可以调用PointPostAction.manualPostPoint(View,Object)，调用该方法后，会直接发送埋点，但是埋点数据的解析会走自动打点的数据配置逻辑。注意：需要调用ignoreAutoTrack、canIgnoreAutoTrack 忽略自动打点对指定控件的支持。

对于完全手动打点的情况（不走埋点配置平台），使用者只需要直接调用 PointPostAction.postNlog()。

## 关联数据注意
对于打点框架支持的控件，底层会通过DataStrategy获取点击操作对应的数据。对于自定义ViewGroup 等不支持自动打点的情况，可以通过
BaseActivity.configLayoutData (id 和数据的对应关系)
AutoTrackHelper.configLayoutData (id 和数据的对应关系)
AutoTrackHelper.ignoreLayoutData
或实现DataAdapter 配置view绑定的数据 (View 实例和数据的对应关系,特别适用于用自定义ViewGroup,通过addView实现list效果的写法)，那么该View下的任意子View点击操作打点日志绑定的数据都是该数据。

## 关于布局文件
布局文件中控件id不能有重复，任意控件id不能和文件名相同，同一个layout.xml中，不同include的布局中不能包含相同id的控件。

## 自定义ViewGroup说明
对于需要绑定数据的自定义ViewGroup,请务必实现DataAdapter 接口，并实现数据绑定方法setData及数据获取方法getData。打点框架会自动获取该自定义布局绑定的数据。

## 关于埋点配置
主项目assets目录下需要放置da.cfg配置文件，这个文件与从服务器下载的配置文件相同，作为默认埋点配置。

## 开发规范的检查已经通过lint项目保证

## License

  Copyright 2017  luoJiSiWei

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
