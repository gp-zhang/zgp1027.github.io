# 下载
    [JCMapKit](https://apis.aliyun183.jcbel.com/file-warehouse/1/JCMapKit/1.1/download)
    [Demo](https://apis.aliyun183.jcbel.com/file-warehouse/1/JCMapKit/1.1/demo)
    [API文档](http://zgp1027.github.io/JCMapKit1.1/html/index.html)
# 工程配置
  1.引入JCMapKit,libc++

  2.在info.plist文件中添加NSLocationWhenInUseUsageDescription:string

  3.
    创建地图配置plist文件,所有字段皆为必填项，按照顺序添加，添加的顺序对应定位相关的楼层索引

            <plist version="1.0">
            <array>
            	<dict>
            		<key>map</key>
            		<string>map(地图文件名，后缀为jmk)</string>
            		<key>name</key>
            		<string>B2(地图名称)</string>
            		<key>num</key>
            		<integer>-2(楼层号)</integer>
            		<key>uuid(对应beacon的uuid)</key>
            		<string>FDA50693-A4E2-4FB1-AFCF-C6EB07647825</string>
            	</dict>
            </array>
            </plist>

  4.引入地图文件

# 加载建筑物(地图数据)

      [[JCMapKit getInstance] loadBuilding:[[NSBundle mainBundle] pathForResource:@"map" ofType:@"plist"]];

# 创建mapView,并显示地图
    JCMapView *mapView = [[JCMapView alloc] initWithFrame:self.view.bounds];
    [mapView loadBuilding:[[JCMapKit getInstance]  getBuilding]];//加载建筑
    [mapView setCurrentFloor:0];//显示指定索引的楼层，以plist文件顺序为准
    mapView.delegate = self;
    [self.view addSubview:mapView];
# POI信息
##  添加Poi数据
    //除LAYER_STALL类型图层外，其余图层都需要设置图片
    [mapView addLayer:LAYER_ELEVATOR image:[UIImage imageWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"marker_wc" ofType:@"png"]]];
    [mapView addLayer:LAYER_STALL image:nil];
### 对LAYER_STALL 图层要素分别设置颜色或者文字
    NSArray *features = [mapView getLayerFeatures:@"stall"];

    for (JCFeature *feature in features) {
        if ([[feature getName] containsString:@"5"]) {
            [mapView setStallColor:feature color:[UIColor greenColor]];
        }
    }
### poi点击回调
    - (void)mapView:(JCMapView *)mapView tappedFeature:(JCFeature *)feature
    {
        self.mapView.calloutView.titleLabel.text = [feature getName];
        [self.mapView showCalloutForPosition:[feature getCenter] offset:CGPointZero animated:YES];
    }

## 添加标注
###    例userAnnotation//用户定位标注
    - (void)updateUserLocation:(JCMapLocation *)location
    {
        if (self.userAnnotation == nil) {
            self.userAnnotation = [JCAnnotation annotationWithPoint:CGPointMake(location.mapX, location.mapY)];
            self.userAnnotation.shouldScaleWithSupper = NO;
            [self.mapView addAnnotation:self.userAnnotation animated:YES];
        }
        self.userAnnotation.point = CGPointMake(location.mapX, location.mapY);
    }
###    任何标注都需要在此方法返回JCAnnotationView实例，否者无法显示
    - (JCAnnotationView *)mapView:(JCMapView *)mapView viewForAnnotation:(JCAnnotation *)annotation{
        if (annotation == self.userAnnotation) {
            JCUserAnnotationView *View = [[JCUserAnnotationView alloc] initWithAnnotation:annotation];
            View.frame = CGRectMake(0, 0, 30, 30);
            return View;
        }
        return nil;
    };
### 标注点击触发
    - (void)mapView:(JCMapView *)mapView tappedOnAnnotation:(JCAnnotation *)annotation
    {
        JCFeature *feature = annotation.dataObject;
        //显示气泡
        [self.mMapView showCalloutForPosition:annotation.point offset:CGPointMake(0, 0) animated:YES];
    }
# 定位
## 开启结束定位
    [[JCMapKit getInstance] setDelegate:self];
    [[JCMapKit getInstance] startUpdateUserLocation];
    [[JCMapKit getInstance] stopUpdateUserLocation];
## 位置更新,通过delegate回调
    - (void)updateUserLocation:(JCMapLocation *)location
    {
        if (self.userAnnotation == nil) {
            self.userAnnotation = [JCAnnotation annotationWithPoint:CGPointMake(location.mapX, location.mapY)];
            self.userAnnotation.shouldScaleWithSupper = NO;
            [self.mapView addAnnotation:self.userAnnotation animated:YES];
        }
        self.userAnnotation.point = CGPointMake(location.mapX, location.mapY);
    }
# 导航
    JCMapLocation *start = [[JCMapLocation alloc] initWithFloor:0 X:self.userAnnotation.point.x Y:self.userAnnotation.point.y];
    JCMapLocation *end = [[JCMapLocation alloc] initWithFloor:0 X:endpoint.x Y:endpoint.y];
    //设置导航路线，使用默认样式则可不设置
    [self.mapView setNavigateChainColor:[UIColor redColor]];
    //第一个拐角箭头颜色，可不设置
    [self.mapView setNavigateRoundCornerColor:[UIColor greenColor]];
    //在mapView中绘制路线
    JCNavigateChain *nc = [[JCMapKit getInstance] getNavigateChain:start end:end];
    [self.mapView setNavigateChain:nc];
## 获取导航提示文字
    - (void)updateHeading:(CLHeading *)newHeading
    {
        JCNavicStatus *status = [self.mapView.navigateChain getStatus:[[JCMapLocation alloc] initWithFloor:0 X:self.userAnnotation.point.x Y:self.userAnnotation.point.y] azimuth:newHeading.magneticHeading];

        NSArray *suggestions = @[@"前行",@"左前行",@"向左行",@"后转调头",@"后转调头",@"后转调头",@"向右行",@"右前行"];
        NSArray *nextsegdirections = @[@"左转",@"右转",@"到达"];

        NSLog(@"%@ %.1f米后 %@",suggestions[status.suggestion],status.disttarget,nextsegdirections[status.nextsegdirection]);;
    }
