## let's start with move security blogs (OtterSec)

- an auditor's introduction: https://github.com/0xriazaka/Move-Audit-Resources?tab=readme-ov-file
    - **summary of this article**: **Move** is a blockchain programming language designed for safer smart contracts. It uses **strong typing** to prevent you from accidentally mixing up different types of tokens/coins (like preventing Bitcoin code from being used with Ethereum), and **formal verification** to mathematically prove your code works correctly. Think of it as "training wheels for blockchain programming" - it catches dangerous mistakes before they happen, but isn't a magic solution to all security problems.
 
- move prover: https://osec.io/blog/2022-09-16-move-prover
    - **summary of this article**: The **Move Prover** is a tool that mathematically proves your blockchain code works correctly by checking your written specifications (rules like "this function never crashes" or "result equals a + b") against **all possible inputs**, not just test cases. You write simple rules in English-like statements, and it uses automated theorem proving to verify your code follows those rules universally. It's particularly useful for proving economic invariants (like "users can't drain funds") and safety properties that traditional testing might miss.
    
- unique aspects of the move: https://x.com/osec_io/status/1641543816581726209

## move security blogs (zellic)
- the billion dollar move bug : https://www.zellic.io/blog/the-billion-dollar-move-bug
    - **summary:** Security researchers found a **critical bug** in Move's bytecode verifier that affected billions of dollars across platforms like Sui and Aptos - for functions with exactly 65,534 instructions, the last instruction's security checks were completely ignored due to an incorrect early return statement. This allowed attackers to bypass Move's core security properties, potentially stealing flash loans, creating multiple references to the same object, and deleting "undeletable" assets. The bug was so severe that all affected blockchain platforms had to secretly coordinate emergency fixes to prevent exploitation.
    
- https://www.zellic.io/blog/top-10-aptos-move-bugs/

- https://www.zellic.io/blog/move-fast-and-break-things-pt-1/

## Diff between aptos move and sui move


### **Storage & Access Model**

**Aptos Move:**
- Uses **global storage** - objects stored at accounts
- Contracts can access ANY object of types they define using `borrow_global`
- One object per type per account
- Functions use `acquires` keyword to declare global storage access

**Sui Move:**
- **No global storage** - objects passed explicitly into functions
- Objects have unique IDs (UIDs) and exist independently  
- Multiple objects of same type can exist per account
- Objects must be passed as function parameters (like Solana's account model)

### **Object Lifecycle**

**Aptos:**
```move
// Access any Coin<T> at any address
let coin = borrow_global<Coin<T>>(some_address);
```

**Sui:**
```move
// Must explicitly pass object by UID
public fun use_coin(coin: Coin<T>) { ... }
```

### Sui's Weird Attack Vectors

### **1. UID Swapping Attack**
```move
public entry fun transmute(cat: Cat, dog: Dog) {
    let Cat { id: cat_id } = cat;  // Extract cat's UID
    let Dog { id: dog_id } = dog;  // Extract dog's UID
    
    // Swap the UIDs!
    let new_cat = Cat { id: dog_id };  // Cat with dog's ID
    let new_dog = Dog { id: cat_id };  // Dog with cat's ID
}
```
**Result**: Object ID `0x123` was a Cat, now it's a Dog with the same ID!

### **2. Object Hiding/Resurrection**
```move
// Hide object (even without `store` ability)
public entry fun entomb(cat: Cat) {
    let Cat { id: cat_id } = cat;  // Extract UID
    let tomb = Tomb { 
        id: object::new(ctx),
        cat_id: cat_id  // Store UID inside tomb
    };
    // Cat object disappears, only tomb exists
}

// Bring it back later
public entry fun resurrect(tomb: Tomb) {
    let Tomb { cat_id, .. } = tomb;
    let cat = Cat { id: cat_id };  // Recreate with same UID
}
```
**Result**: Object appears to be deleted, then magically reappears later!

### **3. Unauthorized Transfers**
**Issue**: Objects with `store` ability can be transferred without creator module's permission
```move
// Anyone can do this if object has `store`
transfer::transfer(valuable_nft, attacker_address);
```

### **4. Unauthorized Freezing**
**Issue**: Anyone can freeze objects with `store` ability, making them globally accessible
```move
// Attacker can freeze your private object
freeze_object(your_private_token);
// Now it's globally readable by everyone!
```

**Bottom Line**: Sui's flexibility creates **identity confusion** - objects can change types, disappear/reappear, and transfer unexpectedly, breaking off-chain tracking systems! 🎭