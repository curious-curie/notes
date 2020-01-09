https://developers.google.com/tag-manager/enhanced-ecommerce

### 상품 상세페이지 화면 (page load)

```html
<script>
// Measure a view of product details. This example assumes the detail view occurs on pageload,
// and also tracks a standard pageview of the details page.
dataLayer.push({
  'ecommerce': {
    'detail': {
      'actionField': {'list': 'Apparel Gallery'},    
      // 'detail' actions have an optional list property.
      'products': [{
        'name': product.title,         // Name or ID is required.
        'id': product.id,
        'price': product.price,
        'brand': product.brand,
        'category': product.category,
        'variant': product.variant
       }]
     }
   }
});
</script>
```

### 장바구니 추가 클릭 시 (click event)

```js
// Measure adding a product to a shopping cart by using an 'add' actionFieldObject
// and a list of productFieldObjects.
dataLayer.push({
  'event': 'addToCart',
  'ecommerce': {
    'currencyCode': 'KOR',
    'add': {                                
      // 'add' actionFieldObject measures.
      'products': [{                        
        //  adding a product to a shopping cart.
        'name': 'Triblend Android T-Shirt',
        'id': '12345',
        'price': '15.25',
        'brand': 'Google',
        'category': 'Apparel',
        'variant': 'Gray',
        'quantity': 1
       }]
    }
  }
});
```

### 장바구니에서 삭제 (click event)

```js
// Measure the removal of a product from a shopping cart.
dataLayer.push({
  'event': 'removeFromCart',
  'ecommerce': {
    'remove': {                               
      // 'remove' actionFieldObject measures.
      'products': [{                          
        //  removing a product to a shopping cart.
          'name': 'Triblend Android T-Shirt',
          'id': '12345',
          'price': '15.25',
          'brand': 'Google',
          'category': 'Apparel',
          'variant': 'Gray',
          'quantity': 1
      }]
    }
  }
});
```

### checkout 버튼 클릭 (click event)

```js
<script>
/**
 * A function to handle a click on a checkout button. This function uses the eventCallback
 * data layer variable to handle navigation after the ecommerce data has been sent to Google Analytics.
 */
function onCheckout() {
  dataLayer.push({
    'event': 'checkout',
    'ecommerce': {
      'checkout': {
        'actionField': {'step': 1, 'option': 'Visa'},
        // step: 결제 여러단계 있을 시 단계 표시
        'products': [{
          'name': 'Triblend Android T-Shirt',
          'id': '12345',
          'price': '15.25',
          'brand': 'Google',
          'category': 'Apparel',
          'variant': 'Gray',
          'quantity': 1
       }]
     }
   },
   'eventCallback': function() {
      document.location = 'checkout.html';
   }
  });
}
</script>
```

### 구매 완료 (구매내역 조회 페이지, page load) 

```js
<script>
// Send transaction data with a pageview if available
// when the page loads. Otherwise, use an event when the transaction
// data becomes available.
dataLayer.push({
  'ecommerce': {
    'purchase': {
      'actionField': {
        'id': 'T12345',                         
        // Transaction ID. Required for purchases and refunds.
        'affiliation': 'Online Store',
        'revenue': '35.43',                    
        // Total transaction value (incl. tax and shipping)
        'tax':'4.90',
        'shipping': '5.99',
        'coupon': 'SUMMER_SALE'
      },
      'products': [{                           
        // List of productFieldObjects.
        'name': 'Triblend Android T-Shirt',     
        // Name or ID is required.
        'id': '12345',
        'price': '15.25',
        'brand': 'Google',
        'category': 'Apparel',
        'variant': 'Gray',
        'quantity': 1,
        'coupon': ''                            
        // Optional fields may be omitted or set to empty string.
       },
       {
        'name': 'Donut Friday Scented T-Shirt',
        'id': '67890',
        'price': '33.75',
        'brand': 'Google',
        'category': 'Apparel',
        'variant': 'Black',
        'quantity': 1
       }]
    }
  }
});
</script>
```



1) https://github.com/nuxt-community/modules/tree/master/packages/google-tag-manager

nuxt google tag manager plugin 이용

`this.$gtm.pushEvent({ event: 'myEvent', ...someAttributes })`

2) `plugins/ga.js` 파일 작성

 https://ko.nuxtjs.org/faq/google-analytics/

https://developers.google.com/analytics/devguides/collection/analyticsjs/enhanced-ecommerce 따라 구현 

```js
// plugins/ga.js 
import router from '~router'
/*
** 클라이언트 사이드와 프로덕션 모드에서만 실행됩니다
*/
if (process.env.NODE_ENV === 'production') {
  /*
  ** Google 애널리틱스 스크립트를 include
  */
  (function (i, s, o, g, r, a, m) {
    i.GoogleAnalyticsObject = r; i[r] = i[r] || function () {
      (i[r].q = i[r].q || []).push(arguments)
    }, i[r].l = 1 * new Date(); a = s.createElement(o),
    m = s.getElementsByTagName(o)[0]; a.async = 1; a.src = g; m.parentNode.insertBefore(a, m)
  })(window, document, 'script', 'https://www.google-analytics.com/analytics.js', 'ga')
  /*
  ** 현재 페이지를 설정
  */
  ga('create', 'UA-XXXXXXXX-X', 'auto')
}

export default ({ app: { router }, store }) => {
  /*
  ** 라우트가 변경될 때마다 실행 (초기 설정 시에도 실행됨)
  */
  router.afterEach((to, from) => {
    /*
    ** Google 애널리틱스에게 페이지뷰가 추가된 것을 전달
    */
    ga('set', 'page', to.fullPath)
    ga('send', 'pageview')
  })
}
```

```js
//nuxt.config.js
module.exports = {
  plugins: [
    { src: '~plugins/ga.js', mode: 'client' }
  ]
}
```

```js
ga('ec:addProduct', {               // Provide product details in a productFieldObject.
  'id': 'P12345',                   // Product ID (string).
  'name': 'Android Warhol T-Shirt', // Product name (string).
  'category': 'Apparel',            // Product category (string).
  'brand': 'Google',                // Product brand (string).
  'variant': 'Black',               // Product variant (string).
  'position': 1,                    // Product position (number).
  'dimension1': 'Member'            // Custom dimension (string).
});

ga('ec:setAction', 'click', {       // click action.
  'list': 'Search Results'          // Product list (string).
});
```



3) analytics-module 사용

`this.$ga.ecommerce.addItem`의 형태로 

## vue-analytics

[Tips & Tricks for vue-analytics](https://medium.com/dailyjs/tips-tricks-for-vue-analytics-87a9d2838915)

https://github.com/MatteoGabriele/vue-analytics

```js
import Vue from 'vue'
import VueAnalytics from 'vue-analytics'

Vue.use(VueAnalytics, {
  id: 'UA-XXX-X',
  ecommerce: {
    enabled: true,
    enhanced: true
  }
})
```

```html
<template>
  <div>
    <button @click="addItem">Add item!</button>
  </div>
</template>

<script>
  export default {
    name: 'myComponent',

    methods: {
      addItem () {
        this.$ga.ecommerce.addItem({
          id: '1234',                     // Transaction ID. Required.
          name: 'Fluffy Pink Bunnies',    // Product name. Required.
          sku: 'DD23444',                 // SKU/code.
          category: 'Party Toys',         // Category or variation.
          price: '11.99',                 // Unit price.
          quantity: '1'                   // Quantity.
        })
      }
    }
  }
```



## nuxt: analytics-module

https://github.com/nuxt-community/analytics-module

vue-analytics options 사용 

```js
export default {
  buildModules: [
    ['@nuxtjs/google-analytics', {
      id: 'UA-12301-2'
    }]
  ]
}
```