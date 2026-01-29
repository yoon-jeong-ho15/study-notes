---
title: RTL 비동기 유틸 함수를 통한 노출 테스트 작성
date: 2025-10-20
order: 6
---
# RTL 비동기 유틸 함수를 통한 노출 테스트 작성

이제 `ProductList` 컴포넌트에서 **데이터를 가져오는 api**(`useProducts`)가 제대로 작동해서 화면에 올바르게 렌더링 되는지 확인하기 위한 코드를 작성한다.

```jsx
//ProductList.spec.jsx
it('로딩이 완료된 경우 상품 리스트가 제대로 모두 노출된다', async () => {
  await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

  const productCards = screen.getByTestId('product-card'); // 사전에 정의함

  expect(productCards).toHaveLength(PRODUCT_PAGE_LIMIT);

  productCards.forEach((el, index) => {
    const productCard = within(el);
    const product = data.products[index];

    expect(productCard.getByText(product.title)).toBeInTheDocument();
    expect(productCard.getByText(product.category.name)).toBeInTheDocument();
    expect(
      productCard.getByText(formatPrice(product.price)),
    ).toBeInTheDocument();
    expect(
      productCard.getByRole('button', { name: '장바구니' }),
    ).toBeInTheDocument();
    expect(
      productCard.getByRole('button', { name: '구매' }),
    ).toBeInTheDocument();
  });
});
```

`getByTestId` : 프로덕트카드 컴포넌트는 지금까지 사용해온 매처 `getByText`나 `getByRole`를 통해 확인하기 힘들다. 
미리 `ProductCard`컴포넌트의 **테스트 아이디를 지정**해서 그것으로 찾을 수 있다. 컴포넌트의 속성에 이렇게 추가해서 정의했

```jsx
//ProductCard.jsx
//...
  return (
    <Grid
      item
      xs={6}
      sm={6}
      md={3}
      onClick={handleClickItem}
      data-testid="product-card" // <- 이것으로 찾아낸다
    >
```

그런데 위의 테스트에는 문제가 있는데, `getByTestId` 를 사용하는 것이다.
테스트 코드는 기본적으로 동기적으로 실행되어 비동기(Promise) 코드는 실행하지 않는다.

### `findby...`  함수

그러나 상품 목록을 가져오는 **API 호출은 비동기 방식으로 작동하고 promise를 반환**하기 때문에 `getBy`로 렌더링 결과를 확인할 수 없다.
대신에 `findBy...` 함수를 사용하면 정해진 시간동안 여러번 재시도를 반복해 api 응답이 완료되어 **완전히 데이터를 가져올 때 까지 기다렸다가** 성공 여부를 확인할 수 있다.

```jsx
it('로딩이 완료된 경우 상품 리스트가 제대로 모두 노출된다', async () => {
  await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

  const productCards = await screen.findAllByTestId('product-card');

  expect(productCards).toHaveLength(PRODUCT_PAGE_LIMIT);
```
## 통합테스트에서의 스파이 함수

단위 테스트에서 스파이 함수를 사용해 **함수의 호출 여부와 전달되는 인자들을 검증**할 수 있었다. 
통합 테스트에서도 스파이 함수를 사용하는데 수업에서 사용한 방식은 다음과 같다.

```jsx
it('장바구니 버튼 클릭시 "장바구니 추가 완료!" toast를 노출하며, addCartItem 메서드가 호출된다.', async () => {
    const addCartItemFn = vi.fn();
    mockUseCartStore({ addCartItem: addCartItemFn });

    const { user } = await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

    await screen.findAllByTestId('product-card');

    // 첫번째 상품을 대상으로 검증한다.
    const productIndex = 0;
    const product = data.products[productIndex];
    await user.click(
      screen.getAllByRole('button', { name: '장바구니' })[productIndex],
    );

    expect(addCartItemFn).toHaveBeenNthCalledWith(1, product, 10, 1);
    expect(
      screen.getByText(`${product.title} 장바구니 추가 완료!`),
    ).toBeInTheDocument();
  });
});
```

`addCartItem`이라는 스토어 액션을 스파이함수로 대체하고 있다. `src/utils/test/mockZustandStore.jsx` 에 주스탠드 모킹 함수를 작성해 두었는데

```Jsx
// ProductList.jsx
const { addCartItem } = useCartStore(state => pick(state, 'addCartItem'));
...

const handleClickCart = (ev, product) => {
	ev.stopPropagation();
	if (isLogin) {
	  addCartItem(product, user.id, 1);
	  toast.success(`${product.title} 장바구니 추가 완료!`, { id: TOAST_ID });
	} else {
	  navigate(pageRoutes.login);
	}
};
```

테스트용으로 렌더링 된 컴포넌트는 저 handleClickCart가 실행될 때  주스탠드에 정의되어 있는 기존의 `addCartItem`이 아닌 스파이 함수를 호출한다.
