pragma solidity ^0.8.4;

// 引入ERC20接口
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

// 实现ERC20接口
contract MyToken is IERC20 {
    // 定义映射存储账户余额
    mapping(address => uint256) private _balances;
    
    // 记录代币总供应量
    uint256 private _totalSupply;
    
    // 映射存储账户的允许额度，用于转账授权
    mapping(address => mapping(address => uint256)) private _allowances;
    
    // 代币名称
    string private _name = "MyToken";
    
    // 代币符号
    string private _symbol = "MTK";
    
    // 小数位数
    uint8 private _decimals = 18;
    
    // 构造函数，初始化代币总供应量和分配给合约创建者的代币
    constructor(uint256 initialSupply) {
        _totalSupply = initialSupply;
        _balances[msg.sender] = initialSupply;
        emit Transfer(address(0), msg.sender, initialSupply);
    }
    
    // 返回代币总供应量
    function totalSupply() public view override returns (uint256) {
        return _totalSupply;
    }
    
    // 返回指定地址的代币余额
    function balanceOf(address account) public view override returns (uint256) {
        return _balances[account];
    }
    
    // 转账函数
    function transfer(address recipient, uint256 amount) public virtual override returns (bool) {
        _transfer(msg.sender, recipient, amount);
        return true;
    }
    
    // 内部转账函数，减少发送者余额，增加接收者余额，并触发Transfer事件
    function _transfer(address sender, address recipient, uint256 amount) internal {
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(_balances[sender] >= amount, "ERC20: transfer amount exceeds balance");
        _balances[sender] -= amount;
        _balances[recipient] += amount;
        emit Transfer(sender, recipient, amount);
    }
    
    // 允许另一个地址代为花费自己的代币
    function approve(address spender, uint256 amount) public virtual override returns (bool) {
        _approve(msg.sender, spender, amount);
        return true;
    }
    
    // 内部批准函数，设置_spender可以从_owner账户转移的代币数量
    function _approve(address owner, address spender, uint256 amount) internal {
        require(owner != address(0), "ERC20: approve from the zero address");
        require(spender != address(0), "ERC20: approve to the zero address");
        _allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    }
    
    // 返回_spender从_owner账户中可转移的代币数量
    function allowance(address owner, address spender) public view virtual override returns (uint256) {
        return _allowances[owner][spender];
    }
    
    // 通过审批过的额度进行转账
    function transferFrom(address sender, address recipient, uint256 amount) public virtual override returns (bool) {
        _transfer(sender, recipient, amount);
        _approve(sender, msg.sender, _allowances[sender][msg.sender] - amount);
        return true;
    }
    
    // 发送Transfer事件
    event Transfer(address indexed from, address indexed to, uint256 value);
    
    // 发送Approval事件
    event Approval(address indexed owner, address indexed spender, uint256 value);
}