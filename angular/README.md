angular 1.2.* 兼容 IE8 

	1. 尽可能减少$apply的使用（在rootScope调用$digest，消耗太大，IE8性能可能存在问题），$timeout亦然（尤其不要在HTTP请求时使用，会导致$digest，致使filter等重复运行)
	2. ng-show 的使用请注意在全局增加
		.ng-hide {
            display: none !important;
        }
        IE8 对于angular的ng-hide selector解析有问题 所以需要重新申明
        
    3. 在跳页尤其是页面自动跳页的时候不要使用 
    	window.location.href = '#/**/**';
    	使用 $location.path('/**/**');    
    	
    	<a href="#" ng-click="addIP(ip)">Add some IP</a>
    	
    	a标签也去除href IE8 解析依然会导致问题原因如下：
    	
    	// update browser
		var changeCounter = 0;
		$rootScope.$watch(function $locationWatch() {
  		var oldUrl = $browser.url();
  		var currentReplace = $location.$$replace;

  		if (!changeCounter || oldUrl != $location.absUrl()) {
    		changeCounter++; /* <-- breakpoint here, line 7650 */
    		$rootScope.$evalAsync(function() {
      		if ($rootScope.$broadcast('$locationChangeStart', $location.absUrl(), oldUrl).
          		defaultPrevented) {
        		$location.$$parse(oldUrl);
      		} else {
        		$browser.url($location.absUrl(), currentReplace);
        		afterLocationChange(oldUrl);
      		}
    		});
  		}
  		$location.$$replace = false;

  		return changeCounter;
		});
		
		如果写以上代码 可能导致IE下 $browser.url()和$location.absUrl()不等 导致watch的循环调用最后报错：IE Error: 10 $digest() iterations reached. Aborting

		4. {
		   key : **     这里的key不能为某些保留字比如：class等
		}  	
		
		