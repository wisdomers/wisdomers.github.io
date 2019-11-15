
## About EASYCLUB

EASYCLUB is just a game for blockchain,see [https://ecgame.github.io](https://ecgame.github.io/).

## Rule analysis

* = 46% static incomes  
* <=46% dynamic incomes  
* 5% recommended champion prize pool  
* 3% platform fee

### Source code analysis
```javascript
library SafeMath {
    
    function mul(uint256 a, uint256 b) 
        internal 
        pure 
        returns (uint256 c) 
    {
        if (a == 0) {
            return 0;
        }
        c = a * b;
        require(c / a == b, "SafeMath mul failed");
        return c;
    }

    function div(uint256 a, uint256 b) 
        internal 
        pure 
        returns (uint256) 
    {
        // assert(b > 0); // Solidity automatically throws when dividing by 0
        uint256 c = a / b;
        // assert(a == b * c + a % b); // There is no case in which this doesn't hold
        return c;
    }

    /**
    * @dev Subtracts two numbers, throws on overflow (i.e. if subtrahend is greater than minuend).
    */
    function sub(uint256 a, uint256 b)
        internal
        pure
        returns (uint256) 
    {
        require(b <= a, "SafeMath sub failed");
        return a - b;
    }

    /**
    * @dev Adds two numbers, throws on overflow.
    */
    function add(uint256 a, uint256 b)
        internal
        pure
        returns (uint256 c) 
    {
        c = a + b;
        require(c >= a, "SafeMath add failed");
        return c;
    }
    
    /**
     * @dev gives square root of given x.
     */
    function sqrt(uint256 x)
        internal
        pure
        returns (uint256 y) 
    {
        uint256 z = ((add(x,1)) / 2);
        y = x;
        while (z < y) 
        {
            y = z;
            z = ((add((x / z),z)) / 2);
        }
    }
    //10000×(1-5%)^180≈0.98
    /**
     * @dev gives square. multiplies x by x
     */
    function sq(uint256 x)
        internal
        pure
        returns (uint256)
    {
        return (mul(x,x));
    }
    
    /**
     * @dev x to the power of y 
     */
    function pwr(uint256 x, uint256 y)
        internal 
        pure 
        returns (uint256)
    {
        if (x==0)
            return (0);
        else if (y==0)
            return (1);
        else 
        {
            uint256 z = x;
            for (uint256 i=1; i < y; i++)
                z = mul(z,x);
            return (z);
        }
    }
}

contract EasyClub is Referrer {

  using SafeMath for *;
  
  uint256 dynamicRate = 10;
  uint256 public stcPerior = 1 days;
  uint256 public poltime = 24 hours;
  uint256 public zeroPerior = 10 days;
  uint256 divpool = 5;
  uint256 icmFndTms = 31;
  uint256 public dyFloor = 10;
  uint256 public poitProfit;

  string constant public name = "Easy Club";
  mapping (uint256 => uint256) id2Mask;
  mapping (address => mapping (uint256 => Data.PlayerRound)) public playerRnds;
  mapping (uint256 => Data.Round) public round;
  mapping (address => Data.Player) public player;

  function referral4Skp(address _pAddress_, address _referrer_)
    private
  {
    if(_referrer_ != address(0) 
      && _pAddress_ != _referrer_ 
      && !isExitsReferrer(_pAddress_) 
      && player[_referrer_].t > 0
      && isCanRegist(_pAddress_)
      && !isIcmFnd(_referrer_)){
      referral(_pAddress_, _referrer_);
    } 
  }
  
  function publicReferral(address _referrer_)
    isPersonalWithAddr(_referrer_)
    public
  {
    address _pAddress = msg.sender;
    require(_referrer_ != address(0) && _pAddress != _referrer_);
    require(!isExitsReferrer(_pAddress));
    require(player[_referrer_].t > 0);
    require(isCanRegist(_pAddress));
    require(!isIcmFnd(_referrer_));
    referral(_pAddress, _referrer_);
    
  }

  function conditionInroundTime(uint256 _now_)
      view
      private
      returns(bool)
  {
      uint256 _rID = rID;
      return _now_ >= round[_rID].strt && (_now_ <= round[_rID].end);
  }

 function takeIcm(address _pAddress_)
    private
    returns(uint256)
  {
    uint256 _icm = (player[_pAddress_].win).add(player[_pAddress_].gen).add(player[_pAddress_].dy);
    uint256 _res;
    if (_icm > 2)
    {
        player[_pAddress_].win = 0;
        player[_pAddress_].gen = 1;
        //default 1
        player[_pAddress_].dy = 1;
        _res = _icm.sub(2);

    }
    return(_res);
  }

  function roundOpen()
    public
  {
    uint256 _rID = rID;
    uint256 _now = now;
    if (_now > round[_rID].end && round[_rID].ended == false)
    {
      round[_rID].ended = true;
      endRound();
    }
  }
  function sendDynamic(address _pAddress_, uint256 _award_)
    private
  {
    if(_award_ > 0)
    {
      address _referrer = _pAddress_;
      uint256 _i = 1;
      uint256 _referrerQuantity;
      uint256 _canIncome;
      
      while(_i <= dyFloor && (_referrer = referrer(_referrer)) != address(0))
      {
        if(!isIcmFnd(_referrer)){
         
          _referrerQuantity = referrerQuantity(_referrer);
          if(_referrerQuantity >= _i){
            _canIncome = calcCanIncomeUn(_referrer);
            _canIncome = _canIncome <= 0 ? 0 :  _canIncome < _award_ ? _canIncome : _award_;
            player[_referrer].dy = _canIncome.add(player[_referrer].dy);
            player[_referrer].tdy = _canIncome.add(player[_referrer].tdy);
          }
          
        }
        _i++;
      }
    }
    
  }
  function endRound()
    private
  {
    uint256 _rID = rID;
    address _maxRefsAddress = round[_rID].maxRefsAddress;
    uint256 _win;
    uint256 _pot = round[_rID].pot;
    if(_pot > 0)
    {
      if(_maxRefsAddress != address(0)){
        _win = (_pot.mul(10)).div(100);
        if(_win > 0){
          player[_maxRefsAddress].win = _win.add(player[_maxRefsAddress].win);
        }
      }
    }
    
    rID++;
    _rID++;
    round[_rID].strt = now;
    round[_rID].end = now.add(poltime);
    round[_rID].pot = _pot.sub(_win);
  }
}


```

## Our open source license


