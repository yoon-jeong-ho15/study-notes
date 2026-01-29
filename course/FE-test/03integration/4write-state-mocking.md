---
title: 통합 테스트 작성 - 상태 관리 모킹
date: 2025-10-19
order: 4
---
# 통합 테스트 작성하기 - 상태 관리 모킹

위에서 말한 두 컴포넌트를 테스트하기 위해 상태 관리를 모킹하는 방법을 알았으니 이제 적용해본다.

## ProductInfoTable

```jsx
const ProductInfoTable = () => {
  const { cart, removeCartItem, changeCartItemCount } = useCartStore(state =>
    pick(state, 'cart', 'removeCartItem', 'changeCartItemCount'),
  );
  const { user } = useUserStore(state => pick(state, 'user'));

  return (
    <TableContainer component={Paper} sx={{ wordBreak: 'break-word' }}>
      <Table aria-label="장바구니 리스트">
        <TableBody>
          {Object.values(cart).map(item => (
            <ProductInfoTableRow
              key={item.id}
              item={item}
              user={user}
              removeCartItem={removeCartItem}
              changeCartItemCount={changeCartItemCount}
            />
          ))}
        </TableBody>
      </Table>
    </TableContainer>
  );
};
```

## 단위 테스트?

하위 컴포넌트인 `ProductInfoTableRow`도 여러 기능(이벤트 핸들러)을 가지고 있는 컴포넌트인데 더 정확한 테스트를 위해서는 이 컴포넌트먼저 테스트를 해야하지 않을까? 아닐까?

강의에서는 어차피 `ProductInfoTableRow`를 검증할때 할 수 있는건 인자로 전달받은 상태나 함수를 제대로 렌더링하고 호출하는지에 대한 여부 밖에 검증할 수 없으니까 통합테스트에 포함하여, 상품 목록을 제대로 출력하는지를 확인하는게 더 효과적이라고 한다.

여기서 아리송해지는게 그러면 애초에 **단위 테스트가 왜 필요한거였더라?** 유틸함수나 커스텀훅 말고 렌더링 컴포넌트를 단위 테스트로 검증 할 필요가 왜 있는거지?
- 찾아보니 여러 곳에서 사용되어 복잡한 조건부 렌더링을 하는 컴포넌트에서는 (`<Button variant={} size={} disabled={} onClick={} ...... />`) 단위 테스트를 하는것이 좋고 그 외의 대부분의 렌더링 컴포넌트들은 사실 통합테스트에 겁증한다고 한다.
- 그래도 여전히 유닛 테스트의 유용성에 대해 의문이 든다.

## 통합 테스트 작성

```jsx
// ProductInfoTable.spec.jsx
beforeEach(() => {
  mockUseUserStore({user: {id: 1}});
  mockUseCartStore({
    cart: {
      6: {
        id: 6,
        title: 'Handmade Cotton Fish',
        price: 809,
        description:
          'The slim & simple Maple Gaming Keyboard from Dev Byte comes with a sleek body and 7- Color RGB LED Back-lighting for smart functionality',
        images: [
          'https://user-images.githubusercontent.com/35371660/230712070-afa23da8-1bda-4cc4-9a59-50a263ee629f.png',
        ],
        count: 3,
      },
      7: {
        id: 7,
        title: 'Awesome Concrete Shirt',
        price: 442,
        description:
          'The Nagasaki Lander is the trademarked name of several series of Nagasaki sport bikes, that started with the 1984 ABC800J',
        images: [
          'https://user-images.githubusercontent.com/35371660/230762100-b119d836-3c5b-4980-9846-b7d32ea4a08f.png',
        ],
        count: 4,
      },
    },
    totalCount: 7,
    totalPrice: 500,
  });
});
```

이렇게 셋업을 통해 상태를 모킹한다.

그리고 다음과 같은 필요한 테스트들을 진행하면 된다.

```jsx
it('장바구니에 포함된 아이템들의 이름, 수량, 합계가 제대로 노출된다', async () => {
  await render(<ProductInfoTable />);

  const [firstItem, secondItem] = screen.getAllByRole('row');

  expect(
    within(firstItem).getByText('Handmade Cotton Fish'),
  ).toBeInTheDocument();
  expect(within(firstItem).getByRole('textbox')).toHaveValue('3');
  expect(within(firstItem).getByText('$2,427.00')).toBeInTheDocument();

  expect(
    within(secondItem).getByText('Awesome Concrete Shirt'),
  ).toBeInTheDocument();
  expect(within(secondItem).getByRole('textbox')).toHaveValue('4');
  expect(within(secondItem).getByText('$1,768.00')).toBeInTheDocument();
});
```

### getBy, queryBy

```jsx
it('특정 아이템의 삭제 버튼을 클릭할 경우 해당 아이템이 사라진다', async () => {
  const { user } = await render(<ProductInfoTable />);
  const [, secondItem] = screen.getAllByRole('row');
  const deleteButton = within(secondItem).getByRole('button');

  expect(screen.getByText('Awesome Concrete Shirt')).toBeInTheDocument();

  await user.click(deleteButton);

  expect(screen.queryByText('Awesome Concrete Shirt')).not.toBeInTheDocument();
});
```

`getByText`는 **DOM에 해당 텍스트가 존재하는지 확인**한다.
그런데 삭제 버튼을 누르면 해당 요소가 사라져야하는데 **사라진것을 확인하려면 `queryBy` 매처**를 사용해야 한다고 한다.

[공식문서](https://testing-library.com/docs/queries/about/)의 설명
- `getBy...`: Returns the matching node for a query, and **throw a descriptive error if no elements match** or if more than one match is found (use `getAllBy` instead if more than one element is expected).
- `queryBy...`: Returns the matching node for a query, **and return `null` if no elements match**. This is useful for asserting an element that is not present. Throws an error if more than one match is found (use `queryAllBy` instead if this is OK).

> 그러면 왜 `queryBy` 로 통일하지 않고 `getBy` 매처도 사용할까?