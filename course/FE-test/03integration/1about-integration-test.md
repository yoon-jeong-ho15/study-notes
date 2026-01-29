---
title: 통합 테스트란
date: 2025-10-19
order: 1
---
# 통합 테스트란?

두 개 이상의 **모듈이 연결된 상태**를 검증.
단위테스트에 비해 **모킹의 비중이 적다**.
모듈들이 상호 작용햐여 발생하는 상태를 검증.
그래서 실제 앱이 동작하는 비즈니스 로직에 가깝게 기능을 검증할 수 있다.

## 통합 테스트 항목

- **특정 상태**를 기준으로 동작하는 컴포넌트 조합
- **API와 함께 상호작용** 하는 컴포넌트 조합
- 단순 UI 렌더링 및 간단한 로직을 실행하는 **컴포넌트(단위 테스트를 하지 않았던)** 가 제대로 렌더링 되는지 한번에 검증

## 예시

수업 자료의 `ProductList`와 `ProductCard` 컴포넌트를 보자.

```jsx
const ProductCard = ({
  product,
  onClickAddCartButton,
  onClickPurchaseButton,
}) => {
 const navigate = useNavigate();

  if (!product) {
    return null;
  }

  const { title, images, price, category, id } = product;

  const handleClickItem = () => {
    navigate(pathToUrl(pageRoutes.productDetail, { productId: id }));
  };
  
  const handleClickAddCartButton = ev => {
    onClickAddCartButton(ev, product);
  };
  
  const handleClickPurchaseButton = ev => {
    onClickPurchaseButton(ev, product);
  };
  
  return (.....);
};
```

프로덕트 카드는 자신을 호출하는 프로덕트리스트 컴포넌트에서 전달받은 prop들(product 객체, 이벤트 핸들러들)을 그대로 사용한다.
- 단위테스트에서는 각 이벤트 핸들러들이 올바른 값을 가지고 호출 되는지 여부만 검증 가능했다.
- 실제 버튼을 눌렀을 때의 **동작은 검증 불가능**했다.
- 여기서 말하는 동작이란 정말 장바구니에 숫자가 늘어나는지 등을 의미

```jsx
const ProductList = ({ limit = PRODUCT_PAGE_LIMIT }) => {
	...
	...
	
	const products =
    data?.pages.reduce((acc, cur) => [...acc, ...cur.products], []) ?? [];

  const handleClickCart = (ev, product) => {
    ev.stopPropagation();
    if (isLogin) {
      addCartItem(product, user.id, 1);
      toast.success(`${product.title} 장바구니 추가 완료!`, { id: TOAST_ID });
    } else {
      navigate(pageRoutes.login);
    }
  };
  
  const handleClickPurchase = (ev, product) => {
    ev.stopPropagation();
    if (isLogin) {
      addCartItem(product, user.id, 1);
      navigate(pageRoutes.cart);
    } else {
      navigate(pageRoutes.login);
    }
  };
	return (
	    <Grid container spacing={1} rowSpacing={1} justifyContent="center">
	      {products.map((product, index) => (
	        <ProductCard
	          key={`${product.id}_${index}`}
	          product={product}
	          onClickAddCartButton={handleClickCart}
	          onClickPurchaseButton={handleClickPurchase}
	        />
			))}
		...
		...
	);
};
```

먼저 `ProductCard`를 *단위테스트* 했을 때 검증할 수 있는 항목들은 다음과 같다 :
1. **prop 기준**으로 제품 정보가(price, name ..) 제대로 렌더링 되는지
2. 상품을 클릭 했을 때(`handleClickItem`) navigate 모킹을 통해 **상세화면으로 이동**하는지
3. 장바구니, 구매 버튼을 눌렀을 때 spy 함수를 통해 **각 핸들러가 호출**되는지
	- 호출이 되는지는 검증할 수 있지만 작동하는지 검증할 수 없다.

반면 `ProductList`를 *통합테스트* 했을 때 검증할 수 있는 항목들은 다음과 같다 :
1. 상품 리스트 조회 **api에 맞게 상품 정보** 가 제대로 렌더링 되는지
2. 상품을 클릭 했을 때 navigate 모킹을 통해 상세화면으로 이동하는지
3. 장바구니 혹은 구매 버튼을 눌렀을 때 **다음과 같이 작동 하는지**
	- 로그인 : 상품 추가 후 장바구니로 이동
	- 비로그인 : 로그인 페이지로 이동
4. 상품 리스트가 더 있는 경우 show more 버튼이 노출되고, 이를 통해 데이터를 더 가져올 수 있는지

위의 두 항목들을 비교해보면 1,2,3번은 **중복**된다고 볼 수 있다. (통1,2,3 이 성공한다는것은 단1,2,3 이 성공함을 **함의**한다) 
그래서 프로덕트 카드에 대한 단위테스트를 하지 않고 프로덕트리스트의 **통합테스트에서 한번에 검증**하는것이 더 효율적이다.
