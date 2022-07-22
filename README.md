# substrate_poe

substrate入门第5课作业，本作业分为两个部分：

1、创建和撤销存证  
2、转移存证


## 第一部分：创建和撤销存证

### 启动链

![](https://github.com/rustbomber/substrate_poe/blob/main/images/start_chain.png)

### 创建存证

![](https://github.com/rustbomber/substrate_poe/blob/main/images/create_poe.png)

### 查询存证

![](https://github.com/rustbomber/substrate_poe/blob/main/images/query_poe.png)


### 删除存证

![](https://github.com/rustbomber/substrate_poe/blob/main/images/remove_poe.png)


### 重复删除存证-抛出错误

![](https://github.com/rustbomber/substrate_poe/blob/main/images/reremove_poe_with_error.png)

### 删除存证后查询

![](https://github.com/rustbomber/substrate_poe/blob/main/images/query_poe_after_remove.png)

## 第二部分：转移存证

### 原用户创建存证

![](https://github.com/rustbomber/substrate_poe/blob/main/images/transfer_create_poe.png)

### 原用户转移存证

![](https://github.com/rustbomber/substrate_poe/blob/main/images/transfer_poe.png)

### 转移后查询存证

存证的用户从 Alice (5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY) 变成了 Bob (5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty)

![](https://github.com/rustbomber/substrate_poe/blob/main/images/transfer_poe.png)

### 转移后，原用户撤销-抛出错误

![](https://github.com/rustbomber/substrate_poe/blob/main/images/remove_with_error.png)

### 主要代码

```rust
...
#[pallet::event] // <-- Step 3. code block will replace this.
#[pallet::generate_deposit(pub(super) fn deposit_event)]
pub enum Event<T: Config> {
    ClaimCreated(T::AccountId, Vec<u8>),
    /// Event emitted when a claim is revoked by the owner. [who, claim]
    ClaimRevoked(T::AccountId, Vec<u8>),
    /// 存证转移事件
    ClaimTransferred(T::AccountId, Vec<u8>),
}

...

#[pallet::weight(10_000)]
pub fn transfer_claim(
    origin: OriginFor<T>,
    proof: Vec<u8>,
    to: T::AccountId,
) -> DispatchResult {
    // 获取原存证的所有者
    let sender = ensure_signed(origin)?;

    // 判断存证是否存在，如果不存在则抛出 NoSuchProof 错误
    ensure!(Proofs::<T>::contains_key(&proof), Error::<T>::NoSuchProof);

    // 取出存证
    let (owner, block_number) = Proofs::<T>::get(&proof);
    ensure!(sender == owner, Error::<T>::NotProofOwner);

    // 删除原所有者的存证
    Proofs::<T>::remove(&proof);

    // 将存证转移给新用户
    Proofs::<T>::insert(&proof, (to, block_number));

    // 触发事件
    Self::deposit_event(Event::ClaimTransferred(sender, proof));

    Ok(())
}
```