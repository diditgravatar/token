# token
Untuk membuat token ERC-20 yang lebih lengkap, Anda bisa menambahkan beberapa fitur tambahan seperti minting (pencetakan token), burning (pemusnahan token), pausing (penonaktifan kontrak sementara), serta fitur untuk mengelola token secara lebih fleksibel. Berikut adalah contoh kontrak token ERC-20 dengan fitur tambahan tersebut:

### **ERC-20 Token dengan Fitur Lengkap**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address recipient, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    event Mint(address indexed to, uint256 amount);
    event Burn(address indexed from, uint256 amount);
    event Paused();
    event Unpaused();
}

contract MyAdvancedToken is IERC20 {
    string public constant name = "MyAdvancedToken";
    string public constant symbol = "MAT";
    uint8 public constant decimals = 18;
    
    uint256 private _totalSupply;
    bool private _paused;
    
    mapping(address => uint256) private _balances;
    mapping(address => mapping(address => uint256)) private _allowances;
    
    address private _owner;

    modifier onlyOwner() {
        require(msg.sender == _owner, "Not the contract owner");
        _;
    }

    modifier whenNotPaused() {
        require(!_paused, "Contract is paused");
        _;
    }

    modifier whenPaused() {
        require(_paused, "Contract is not paused");
        _;
    }

    constructor(uint256 initialSupply) {
        _owner = msg.sender;
        _totalSupply = initialSupply * 10 ** uint256(decimals);
        _balances[msg.sender] = _totalSupply;
        _paused = false;
        emit Transfer(address(0), msg.sender, _totalSupply);
    }

    function totalSupply() public view override returns (uint256) {
        return _totalSupply;
    }

    function balanceOf(address account) public view override returns (uint256) {
        return _balances[account];
    }

    function transfer(address recipient, uint256 amount) public override whenNotPaused returns (bool) {
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(_balances[msg.sender] >= amount, "ERC20: insufficient balance");

        _balances[msg.sender] -= amount;
        _balances[recipient] += amount;
        
        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    function allowance(address owner, address spender) public view override returns (uint256) {
        return _allowances[owner][spender];
    }

    function approve(address spender, uint256 amount) public override whenNotPaused returns (bool) {
        require(spender != address(0), "ERC20: approve to the zero address");

        _allowances[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(address sender, address recipient, uint256 amount) public override whenNotPaused returns (bool) {
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(_balances[sender] >= amount, "ERC20: insufficient balance");
        require(_allowances[sender][msg.sender] >= amount, "ERC20: transfer amount exceeds allowance");

        _balances[sender] -= amount;
        _balances[recipient] += amount;
        _allowances[sender][msg.sender] -= amount;

        emit Transfer(sender, recipient, amount);
        return true;
    }

    // Minting: Pencetakan token baru
    function mint(address to, uint256 amount) public onlyOwner {
        _totalSupply += amount;
        _balances[to] += amount;
        emit Mint(to, amount);
    }

    // Burning: Pemusnahan token
    function burn(uint256 amount) public {
        require(_balances[msg.sender] >= amount, "ERC20: insufficient balance to burn");

        _balances[msg.sender] -= amount;
        _totalSupply -= amount;
        emit Burn(msg.sender, amount);
    }

    // Pausing contract: Menonaktifkan sementara transaksi
    function pause() public onlyOwner whenNotPaused {
        _paused = true;
        emit Paused();
    }

    // Unpausing contract: Mengaktifkan kembali transaksi
    function unpause() public onlyOwner whenPaused {
        _paused = false;
        emit Unpaused();
    }
}
```

### **Penjelasan Fitur-fitur Tambahan:**

1. **Minting (Pencetakan Token)**:
   - Fungsi `mint(address to, uint256 amount)` memungkinkan pemilik kontrak untuk mencetak token baru dan mengirimkannya ke alamat tertentu.
   - Hanya pemilik kontrak yang dapat menggunakan fungsi ini (`onlyOwner`).

2. **Burning (Pemusnahan Token)**:
   - Fungsi `burn(uint256 amount)` memungkinkan pengguna untuk memusnahkan token mereka sendiri, mengurangi total supply token.
   - Ini memungkinkan pengguna untuk menghancurkan sebagian token mereka sebagai bagian dari kebijakan deflasi atau untuk berbagai tujuan lainnya.

3. **Pausing (Menonaktifkan Kontrak)**:
   - Fungsi `pause()` dan `unpause()` digunakan untuk menonaktifkan sementara seluruh kontrak (misalnya, dalam situasi darurat atau selama audit keamanan).
   - Saat kontrak dihentikan, transaksi seperti transfer atau approval tidak akan diperbolehkan.

4. **Modifikasi dengan Modifier**:
   - **onlyOwner**: Memastikan hanya pemilik kontrak yang dapat melakukan minting atau pengaturan lainnya.
   - **whenNotPaused** dan **whenPaused**: Untuk mengontrol apakah fungsi tertentu dapat dijalankan berdasarkan status paus kontrak.

### **Event Baru**:
- `Mint`: Dipancarkan ketika token baru dicetak.
- `Burn`: Dipancarkan ketika token dihancurkan.
- `Paused` dan `Unpaused`: Dipancarkan ketika kontrak dijeda atau dilanjutkan.

### **Langkah Selanjutnya:**
- **Deploy ke Testnet**: Anda bisa menguji kontrak ini di testnet Ethereum (seperti Rinkeby atau Ropsten) untuk melihat bagaimana token berfungsi.
- **Menghubungkan ke Frontend**: Gunakan library seperti **Web3.js** atau **Ethers.js** untuk berinteraksi dengan kontrak ini melalui antarmuka pengguna.

Jika Anda ingin tahu lebih lanjut tentang penerapan dan penggunaan lebih lanjut, atau jika Anda ingin menguji kontrak ini di jaringan testnet, saya bisa membantu lebih lanjut.
