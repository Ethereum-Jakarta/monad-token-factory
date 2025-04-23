# Smart Contract Token Factory Monad

Repository ini berisi smart contract untuk membuat dan mengelola token ERC20 di jaringan blockchain Monad.

## Struktur Proyek

```
contracts/
├── Token.sol          # Implementasi token ERC20 dengan fitur burning
└── TokenFactory.sol   # Factory untuk menciptakan token
```

## Smart Contract

### Token.sol

Token.sol adalah implementasi token ERC20 dengan fitur pembakaran (burning):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract Token is ERC20, Ownable {
    constructor(address initialOwner, uint256 initialSupply, string memory tokenName, string memory tokenSymbol) 
    ERC20(tokenName, tokenSymbol)
    Ownable(initialOwner)
    {
        _transferOwnership(initialOwner);
        _mint(initialOwner, initialSupply * 10 ** decimals());
    }

    function burnToken(uint256 burnAmount) public onlyOwner {
        require(balanceOf(msg.sender) >= burnAmount * 10 ** decimals(), "Error : you need more amount");
        _burn(msg.sender, burnAmount * 10 ** decimals());
    }
}
```

**Penjelasan:**
- Mewarisi dari kontrak OpenZeppelin ERC20 dan Ownable
- Constructor menerima parameter:
  - `initialOwner`: Alamat pemilik token
  - `initialSupply`: Jumlah awal token yang dicetak
  - `tokenName`: Nama lengkap token
  - `tokenSymbol`: Simbol token (biasanya 3-4 huruf)
- Fungsi `burnToken`: Memungkinkan pemilik token membakar (menghapus) sejumlah token

### TokenFactory.sol

TokenFactory.sol adalah kontrak factory yang memungkinkan pembuatan token ERC20 dinamis:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "./Token.sol";

contract TokenFactory {

    address[] public createdTokens;
    event createTokenEvent(address indexed owner, address indexed tokenAddress, uint256 totalSupply);

    function createToken(address initialOwner, uint256 
    initialSupply, string memory tokenName, string memory tokenSymbol)
    public returns (address){
        
        Token newToken = new Token(initialOwner, initialSupply, tokenName, tokenSymbol);
        createdTokens.push(address(newToken));

        emit createTokenEvent(initialOwner, address(newToken), initialSupply);
        return address(newToken);
    }

    function getAllTokens() public view returns(address[] memory){
        return createdTokens;
    }

    function getTokensCount() public view returns(uint256) {
        return createdTokens.length;
    }
}
```

**Penjelasan:**
- `createdTokens`: Array untuk menyimpan alamat semua token yang telah dibuat
- `createTokenEvent`: Event yang dipancarkan saat token baru dibuat
- `createToken`: Fungsi utama untuk membuat token baru dengan parameter kustom
- `getAllTokens`: Mengembalikan daftar alamat semua token yang telah dibuat
- `getTokensCount`: Mengembalikan jumlah total token yang telah dibuat

## Memulai Pengembangan

### Prasyarat

- Node.js v18 ke atas
- Hardhat
- OpenZeppelin Contracts

### Instalasi

1. Clone repositori:
   ```bash
   git clone https://github.com/username/monad-token-factory.git
   cd monad-token-factory
   ```

2. Instal dependensi:
   ```bash
   npm install
   ```

3. Kompilasi kontrak:
   ```bash
   npx hardhat compile
   ```

### Testing

Jalankan test suite untuk memverifikasi fungsionalitas kontrak:

```bash
npx hardhat test
```

### Deployment

Deploy TokenFactory ke Monad Testnet:

```bash
npx hardhat run scripts/deploy-token-factory.ts --network monadTestnet
```

## Interaksi dengan Kontrak

### Membuat Token Baru

Setelah TokenFactory di-deploy, gunakan script berikut untuk membuat token baru:

```bash
npx hardhat run scripts/create-new-token.ts --network monadTestnet
```

### Interaksi dengan Token

Untuk berinteraksi dengan token yang sudah dibuat:

```bash
npx hardhat run scripts/interact-token.ts --network monadTestnet
```

## Konfigurasi Hardhat

File konfigurasi Hardhat (`hardhat.config.ts`) harus mengatur jaringan Monad Testnet:

```typescript
monadTestnet: {
  url: "https://testnet-rpc.monad.xyz/",
  chainId: 10143,
  accounts: vars.has("PRIVATE_KEY") ? [`0x${vars.get("PRIVATE_KEY")}`] : [],
  gasPrice: "auto",
}
```

## Keamanan

- Kontrak Token menggunakan OpenZeppelin untuk keamanan maksimal
- Fungsi burning hanya dapat dipanggil oleh pemilik
- Semua token yang dibuat dilacak oleh TokenFactory

## Lisensi

Proyek ini dilisensikan di bawah Lisensi MIT.