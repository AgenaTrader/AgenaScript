# Strategy Programming

## Account
### Description
Account is an object containing information about the account with which the current strategy is working.

The individual properties are:

-   **Account.AccountConnection**
    Name for the broker connection used (the name assigned under the account connection submenu)

-   **Account.AccountType**
    Type of account (live account, simulated account etc.)

-   **Account.Broker**
    Name/definition for the broker

-   **Account.BuyingPower**
    The current account equity in consideration of the leverage provided by the broker (IB leverages your account equity by a factor of 4, meaning that with 10000€ your buying power is equal to 40000€)

-   **Account.CashValue**
    Amount (double)

-   **Account.Currency**
    Currency in which the account is held

-   **Account.ExcessEquity**
    Excess

-   **Account.InitialMargin**
    Initial margin (depends on the broker, double)

-   **Account.InstrumentType**
    Type of trading instrument (type AgenaTrader.Plugins.InstrumentTypes)

-   **Account.IsDemo**
    True, if the account is a demo account

-   **Account.Name**
    Name of the account (should be identical to Account.AccountConnection)

-   **Account.OverNightMargin**
    Overnight margin (depends on the broker, double)

-   **Account.RealizedProfitLoss**
    Realized profits and losses (double)

### Example
```cs
Print("AccountConnection " + Account.AccountConnection);
Print("AccountType " + Account.AccountType);
Print("Broker " + Account.Broker);
Print("BuyingPower " + Account.BuyingPower);
Print("CashValue " + Account.CashValue);
Print("Currency " + Account.Currency);
Print("ExcessEquity " + Account.ExcessEquity);
Print("InitialMargin " + Account.InitialMargin);
Print("InstrumentTypes " + Account.InstrumentTypes);
Print("IsDemo " + Account.IsDemo);
Print("Name " + Account.Name);
Print("OverNightMargin " + Account.OverNightMargin);
Print("RealizedProfitLoss " + Account.RealizedProfitLoss);
```

## BarsCountFromTradeClose()
### Description
The property "BarsCountFromTradeClose" outputs the number of bars that have occurred since the last exit from the market.

### Usage
```cs
BarsCountFromTradeClose()
BarsCountFromTradeClose(string strategyName)
```

For multi-bar strategies
```cs
BarsCountFromTradeClose(int multibarSeriesIndex, string strategyName, int exitsAgo)
```

### Parameter
|                     |                                                                                                                           |
|---------------------|---------------------------------------------------------------------------------------------------------------------------|
| strategyName          | The Strategy name (string) that has been used to clearly label the exit within the exit method.    |
| multibarSeriesIndex | For *[Multibar*](#multibar)[*MultiBars*](#multibars) strategies. Index of the data series for which the exit order has been executed. See [*ProcessingBarSeriesIndex*](#processingbarseriesindex). |
| exitsAgo            | Number of exits that have occurred in the past. A zero indicates the number of bars that have formed after the last exit. |

### Example
```cs
Print("The last exit was " + BarsCountFromTradeClose() + " bars ago.");
```

## BarsCountFromTradeOpen()
### Description
The property "BarsCountFromTradeOpen" returns the number of bars that have occurred since the last entry into the market.

### Usage
```cs
BarsCountFromTradeOpen()
BarsCountFromTradeOpen(string strategyName)
```

For multi-bar strategies

```cs
BarsCountFromTradeOpen(int multibarSeriesIndex, string strategyName, int entriesAgo)
```

### Parameter
|                     |                                                                                                           |
|---------------------|-----------------------------------------------------------------------------------------------------------|
| strategyName          | The strategy name (string) that has been used to clearly label the entry within an entry method.            |
| multibarSeriesIndex | For *[Multibar*](#multibar), [*MultiBars*](#multibars) strategies. Index for the data series for which the entry order was executed. See [*ProcessingBarSeriesIndex*](#processingbarseriesindex), [*ProcessingBarSeriesIndex*](#processingbarseriesindex). |
| entriesAgo          | Number of entries in the past. A zero indicates the number of bars that have formed after the last entry. |

### Example
```cs
Print("The last entry was " + BarsCountFromTradeOpen() + " bars ago.");
```

## CancelAllOrders()
### Description
CancelAllOrders deletes all oders (cancel) managed by the strategy.
A cancel request is sent to the broker. Whether an or there is really deleted, can not be guaranteed. It may happen that an order has received a partial execution before it is deleted.
Therefore we recommend that you check the status of the order with [*OnOrderChanged()*](#onorderchanged).

### Usage
```cs
CancelAllOrders()
```
### Parameter
None

### Example
```cs
protected override void OnCalculate()
{
   if (BarsCountFromTradeOpen() >= 30)
       CancelAllOrders();
}
```

## Order.Cancel()
### Description
Cancel order deletes an order.

A cancel request is sent to the broker. There is no guarantee that the order will actually be deleted there. It may occur that the order receives a partial execution before it is deleted. Therefore we recommend that you check the status of the order with [*OnOrderChanged()*](#onorderchanged).

### Usage
```cs
Order.Cancel(IOrder order)
```

### Parameter
An order object of the type "IOrder"

### Example
```cs
private IOrder entryOrder = null;
private int barNumber = 0;
protected override void OnCalculate()
{
    // Place an entry stop at the high of the current bar
    if (entryOrder == null)
    {
        entryOrder = OpenLongStop(High[0], "stop long");
        barNumber = ProcessingBarIndex;
    }
    // Delete the order after 3 bars
    if (Position.PositionType == PositionType.Flat &&
    ProcessingBarIndex > barNumber + 3)
        Order.Cancel(entryOrder);
}
```


## CreateIfDoneGroup()
### Description
If two orders are linked to one another via a CreateIfDoneGroup, it means that if the one order has been executed, the second linked order is activated.

### Usage
```cs
CreateIfDoneGroup(IEnumerable<IOrder> orders)
```

### Parameter
An order object of type IOrder as a list

### Example
```cs
private IOrder oopenlong = null;
private IOrder osubmitbuy = null;


protected override void OnInit()
{
   IsAutoConfirmOrder = false;
}


protected override void OnCalculate()
{
 
 oopenlong =  SubmitOrder(new StrategyOrderParameters
                {
                    Direction = OrderDirection.Buy,
                    Type = OrderType.Market,
                    Quantity = DefaultOrderQuantity,
                    SignalName = "strategyName",
                });
   
osubmitbuy =  SubmitOrder(new StrategyOrderParameters
                {
                    Direction = OrderDirection.Sell,
                    Type = OrderType.Stop,
                    Quantity = DefaultOrderQuantity,
		    StopPrice = Close[0] * 1.1,
                    SignalName = "strategyName",
                });
				
   CreateIfDoneGroup(new List<IOrder> { oopenlong, osubmitbuy });

   oopenlong.ConfirmOrder();
}

```

## CreateOCOGroup()
### Description
If two orders are linked via a CreateOCOGroup, it means that once the one order has been executed, the second linked order is deleted.

### Usage
```cs
CreateOCOGroup(IEnumerable<IOrder> orders)
```

### Parameter
An order object of type IOrder as a list

### Example
```cs
private IOrder oopenlong = null;
private IOrder oEnterShort = null;


protected override void OnInit()
{
   IsAutoConfirmOrder = false;
}


protected override void OnCalculate()
{
    
oopenlong =  SubmitOrder(new StrategyOrderParameters
                {
                    Direction = OrderDirection.Buy,
                    Type = OrderType.Stop,
                    Quantity = DefaultOrderQuantity,
		    StopPrice = Close[0] * 1.1,
                    SignalName = "strategyName",
                });

  
oEnterShort =  SubmitOrder(new StrategyOrderParameters
                {
                    Direction = OrderDirection.Sell,
                    Type = OrderType.Stop,
                    Quantity = DefaultOrderQuantity,
		    StopPrice = Close[0] * -1.1,
                    SignalName = "strategyName",
                });
				
  
  
   CreateOCOGroup(new List<IOrder> { oopenlong, oEnterShort });

   oopenlong.ConfirmOrder();
   oEnterShort.ConfirmOrder();
}
```

## CreateOROGroup()
### Description
If two orders are linked via a CreateOROGroup, it means that once the one order has been executed, the order size of the second order is reduced by the order volume of the first order.
### Usage
```cs
CreateOROGroup(IEnumerable<IOrder> orders)
```

### Parameter
An order object of type IOrder as a list

### Example
```cs
private IOrder oStopLong = null;
private IOrder oLimitLong = null;


protected override void OnInit()
{
   IsAutoConfirmOrder = false;
}


protected override void OnCalculate()
{
   
   oStopLong =  SubmitOrder(new StrategyOrderParameters
                {
                    Direction = OrderDirection.Buy,
                    Type = OrderType.Stop,
                    Quantity = DefaultOrderQuantity,
		    StopPrice = Close[0] * -1.1,
                    SignalName = "strategyName",
                });
   
   
   
   
   oLimitLong =  SubmitOrder(new StrategyOrderParameters
                {
                    Direction = OrderDirection.Buy,
                    Type = OrderType.Limit,
                    Quantity = DefaultOrderQuantity*0.5,
		    Price = Close[0] * 1.1,
                    SignalName = "strategyName",
                });

   CreateOROGroup(new List<IOrder> { oLimitLong, oStopLong });
}
```


## DataSeriesConfigurable
## DefaultOrderQuantity
### Description
Change order changes an order.

Default quantity defines the amount to be used in a strategy. Default quantity is set within the [*OnInit()*](#oninit) method.

### Usage
```cs
ReplaceOrder(IOrder iOrder, int quantity, double limitPrice, double stopPrice)
```

### Parameter
An int value containing the amount (stocks, contracts etc.)

### Example
```cs
protected override void OnInit()
{
DefaultOrderQuantity = 100;
}
```

## EntriesPerDirection
### Description
Entries per direction defines the maximum number of entries permitted in one direction (long or short).

Whether the name of the entry signal is taken into consideration or not is defined.

Entries per direction is defined with the [*OnInit()*](#oninit) method.

### Usage
**EntriesPerDirection**

### Parameter
An int value for the maximum entries permitted in one direction.

### Example
```cs
// Example 1
// If one of the two entry conditions is true and a long position is opened, then the other entry signal will be ignored
protected override void OnInit()
{
EntriesPerDirection = 1;

}

protected override void OnCalculate()
{
    if (CrossAbove(EMA(14), SMA(50), 1) && IsSerieRising(ADX(20)))
        OpenLong("SMA cross entry");
}

// Example 2


protected override void OnCalculate()
{
    if (CrossAbove(EMA(14), SMA(50), 1) && IsSerieRising(ADX(20)))
        OpenLong("EMACrossesSMA");
    else if (CrossAbove (MACD(2,2,5), 0, 1))
        OpenLong("MACDCross");
}
```

## ExcludeTradeHistoryInBacktest
## CloseLongTrade ()
### Description
CloseLongTrade creates a sell order for closing a long position (sell).

See: [*SubmitOrder()](#submitorder), [*CloseShortTrade()](#closeshorttrade)
### Usage
See [*StrategyOrderParameters*](#strategyorderparameters)

### Parameter
See [*StrategyOrderParameters*](#strategyorderparameters)

### Return Value
An order object of the type "IOrder"

### Example
```cs
var order = CloseLongTrade(new StrategyOrderParameters
{
    Type = OrderType.Market
});
```

## ExitOnClose
## ExitOnCloseSeconds

## CloseShortTrade()
### Description
CloseShortTradecreates a buy-to-cover order for closing a short position (buy).


See: [*SubmitOrder()*](#submitorder), [*CloselongTrade()*](#closelongtrade)
### Usage
See [*StrategyOrderParameters*](#strategyorderparameters)

### Parameter
See [*StrategyOrderParameters*](#strategyorderparameters)

### Return Value
An order object of the type "IOrder"

### Example
```cs
var order = CloseShortTrade(new StrategyOrderParameters
{
    Type = OrderType.Stop,
    Quantity = quantity,
    StopPrice = price
});
```

## Account.GetValue()
### Description
Get account value outputs information regarding the account for which the current strategy is being carried out.

See [*GetProfitLoss()*](#getprofitloss).

### Usage
```cs
Account.GetValue(AccountItem accountItem)
```

### Parameter
Possible values for account item are:

AccountItem.BuyingPower

AccountItem.CashValue

AccountItem.RealizedProfitLoss

### Return Value
A double value for the account item for historical bars, a zero (0) is returned

### Example
```cs
Print("The current account cash value is " + Account.GetValue(AccountItem.CashValue));
Print("The current account cash value with the leverage provided by the broker is " + Account.GetValue(AccountItem.BuyingPower));
Print("The current P/L already realized is " + Account.GetValue(AccountItem.RealizedProfitLoss));
```

## GetEntries()
### Description
This DataSeries is used in conditions and indicates multiple entry prices for entry orders

### Usage
Overload in scripted condition for short and long signal indication

### Parameter
None

### Return Value
int

### Example
```cs
public class MyTestEntry : UserScriptedCondition
	{

        double _percentage = 100;

        protected override void Initialize()
		{
			IsEntry = true;
			IsStop = false;
			IsTarget= false;
			Add(new OutputSeriesDescription(Color.FromKnownColor(KnownColor.Black), "Occurred"));
			Add(new OutputSeriesDescription(Color.FromArgb(255, 118, 222, 90), "Entry1"));
            Add(new OutputSeriesDescription(Color.FromArgb(255, 118, 222, 90), "Entry2"));
            Add(new OutputSeriesDescription(Color.FromArgb(255, 118, 222, 90), "Entry3"));
            Overlay = true;
			CalculateOnBarClose = true;
		}

		protected override void OnBarUpdate()
		{

            Calculate();

		}

        public override void Recalculate()
        {
            Calculate();
        }

        private void Calculate ()
        {

            if (TradeDirection == PositionType.Long)
            {
                Entry1.Set(Close[0] + 0.5);
                Entry2.Set(Close[0] + 1);
                Entry3.Set(Close[0] + 1.5);
            }
            else
            {
                Entry1.Set(Close[0] - 0.5);
                Entry2.Set(Close[0] - 1);
                Entry3.Set(Close[0] - 1.5);
            }
        }

		#region Properties

		[Browsable(false)]
		[XmlIgnore()]
		public DataSeries Occurred
		{
			get { return Values[0]; }
		}

		[Browsable(false)]
		[XmlIgnore()]
		public DataSeries Entry1
		{
			get { return Values[1]; }
		}

        [Browsable(false)]
        [XmlIgnore()]
        public DataSeries Entry2
        {
            get { return Values[2]; }
        }

        [Browsable(false)]
        [XmlIgnore()]
        public DataSeries Entry3
        {
            get { return Values[3]; }
        }

        public override IList<DataSeries> GetEntrys()
		{
            return new[] { Entry1, Entry2, Entry3 };

```

## GetProfitLoss()
### Description
Get profit loss outputs the currently unrealized profit or loss for a running position.

See [*Account.GetValue()*](#accountgetvalue).

### Usage
```cs
GetProfitLoss(int pLType);
```

### Parameter
Potential values for the P/L type are:

0 – Amount: P/L as a currency amount

1 – Percent: P/L in percent

2 – Risk: P/L in Van Tharp R-multiples [*www.vantharp.com*](http://www.vantharp.com/tharp-concepts/risk-and-r-multiples.asp)

3 – P/L in ticks

### Return Value
A double value for the unrealized profit or loss

### Example
```cs
Print("The current risk for the strategy " + this.Name + " is " + GetProfitLoss(1) + " " + Instrument.Currency);
Print("This equals "+ string.Format( "{0:F1} R.", GetProfitLoss(3)));
```

## GetProfitLossAmount()
### Description
GetProfitLossAmount () provides the current unrealized gain or loss of a current position as the currency amount.

See [*Account.GetValue()*](#accountgetvalue).

### Usage
```cs
GetProfitLossAmount(double profitLoss);
```

### Parameter
Double

### Return Value
A double value for the unrealized profit or loss

### Example
```cs
Print("the current P&L " + this.Name + " is " + GetProfitLossAmount(Position.OpenProfitLoss) + " " + Instrument.Currency);
```

## GetProfitLossRisk()
### Description
GetProfitLossRisk () returns the current unrealized gain or loss of a current position in R-multiples.

See [*Account.GetValue()*](#accountgetvalue).

### Usage
```cs
GetProfitLossRisk();
```

### Parameter
None
### Return Value
A double value for the R-Multiple

### Example
```cs
Print("the current P&L " + this.Name + " is " + string.Format( "{0:F1} R.", GetProfitLossRisk()));
```

## GetScriptedCondition()
### Description
This method allows user to communicate between scripts.



## IsAutoConfirmOrder
### Description
IsAutoConfirmOrder determines whether orders are activated automatically. IsAutoConfirmOrder is specified in the [*OnInit()*](#oninit) method.

If IsAutoConfirmOrder = true, then orders are automatically activated (default). If IsAutoConfirmOrder is assigned the value false, the corresponding order must be activated with order.[*ConfirmOrder()*](#confirmorder).

### Parameter
Bool value

### Example
```cs
protected override void OnInit()
{
   IsAutoConfirmOrder = false;
}
```

## Order
### Description
IOrder is an object that contains information about an order that is currently managed by a strategy.


The individual properties are:

-   Action
    **One of four possible positions in the market:**
    -   OrderDirection.Buy
    -   OrderDirection.Sell

-   **AveragePrice**
    **The average purchase or selling price of a position.For positions without partial executions, this corresponds to the entry price.**

-   **FilledQuantity**
    For partial versions

-   **LimitPrice**

-   **Name**
    The unique SignalName  (maybe mistake SignalName)

-   **OrderId**
    The unique OrderId

-   **OrderMode**
    One of three possible positions in the market:
    -   OrderMode.Direct
    -   OrderMode.Dynamic
    -   OrderMode.Synthetic

-   **OrderState**
    The current status of the order can be queried (see *OnOrderExecution* and *OnOrderChanged*)
    -   OrderState.Accepted
    -   OrderState.Cancelled
    -   OrderState.CancelRejected
    -   OrderState.FilledQuantity
    -   OrderState.PartFilled
    -   OrderState.PendingCancel
    -   OrderState.PendingReplace
    -   OrderState.PendingSubmit
    -   OrderState.Rejected
    -   OrderState.ReplaceRejected
    -   OrderState.Unknown
    -   OrderState.Working

-   **OrderType**
    Possible order types:
    -   OrderType.Limit
    -   OrderType.Market
    -   OrderType.Stop
    -   OrderType.StopLimit

-   **Quantity**
    The quantity to be ordered

-   **StopPrice**

-   **Timestamp**
    Time stamp

-   **TimeFrame**
    The TimeFrame, which is valid for the order.

-   **TimeFrame**

Possible Methods:

-   **order Order.Cancel()**
    Delete the Order

-   **order.ConfirmOrder()**
    Confirm the order. This method have to be executed if IsAutoConfirmOrder is set to false and you want to run the order automatically. This is, for example, the case when an OCO or IfDone fabrication is to be produced.

## Performance
### Description
Performance is an object containing information regarding all trades that have been generated by a strategy.

The trades are sorted into multiple lists. With the help of these lists it is easier to create a performance evaluation.

See Performance Characteristics.

The individual lists are:

-   **Performance.AllTrades**
    A [*Trade*](#trade) collection object containing all trades generated by a strategy.

-   **Performance.LongTrades**
    A [*Trade*](#trade) collection object containing all long trades generated by a strategy.

-   **Performance.ShortTrades**
    A [*Trade*](#trade) collection object containing all short trades generated by a strategy.

-   **Performance.WinningTrades**
    A [*Trade*](#trade) collection object containing all profitable trades generated by a strategy.

-   **Performance.LosingTrades**
    A [*Trade*](#trade) collection object containing all loss trades generated by a strategy.

### Example
```cs
// When exiting a strategy, create a performance evaluation
protected override void OnDispose()
{
Print("Performance evaluation of the strategy : " + this.Name);
Print("----------------------------------------------------");
Print("Amount of all trades: " + Performance.AllTrades.Count);
Print("Amount of winning trades: " + Performance.WinningTrades.Count);
Print("Amount of all loss trades: " + Performance.LosingTrades.Count);
Print("Amount of all long trades: " + Performance.LongTrades.Count);
Print("Amount of short trades: " + Performance.ShortTrades.Count);
Print("Result: " + Account.RealizedProfitLoss + " " + Account.Currency);
}
```

## Position
### Description
Position is an object containing information regarding the position currently being managed by a strategy.

The individual properties are:

-   **Position.AvgPrice**
    The average buy or sell price of a position.
    For positions without partial executions, this is equal to the entry price.

-   **Position.CreatedDateTime**
    Date and time at which the position was opened.

-   **Position.Instrument**
    The trading instrument in which the position exists.
    See *Instruments*.

-   **Position.PositionType**
    One of three possible positions in the market:
    -   PositionType.Flat
    -   PositionType.Long
    -   PositionType.Short

-   **Position.OpenProfitLoss**
    The currently not yet realized profit or loss.
    See [*GetProfitLoss()*](#getprofitloss).

-   **Position.ProfitCurrency**
    Profit (or loss) displayed as a currency amount.

-   **Position.ProfitPercent**
    Profit (or loss) displayed in percent.

-   **Position.ProfitPoints**
    Profit (or loss) displayed in points or pips.

-   **Position.Quantity**
    Amount of stocks, contracts, CFDs etc. within a position.

### Example
```cs
if (Position.PositionType != PositionType.Flat)
{
Print("Average price " + Position.AvgPrice);
Print("Opening time " + Position.CreatedDateTime);
Print("Instrument " + Position.Instrument);
Print("Current positioning " + Position.PositionType);
Print("Unrealized P/L " + Position.OpenProfitLoss);
Print("P/L (currency) " + Position.ProfitCurrency);
Print("P/L (in percent) " + Position.ProfitPercent);
Print("P/L (in points) " + Position.ProfitPoints);
Print("Pieces " + Position.Quantity);
}
```

## PositionType
See [*Position.PositionType*](#positionpositiontype).

## TraceOrders
### Description
The trace orders property is especially useful for keeping track of orders generated by strategies. It also provides an overview of which orders were generated by which strategies.
Trace orders can be specified with the [*OnInit()*](#oninit) method.

When TraceOrders is activated, each order will display the following values in the output window:

-   Instrument
-   Time frame
-   Action
-   Type
-   Limit price
-   Stop price
-   Quantity
-   Name

This information is useful when creating and debugging strategies.

### Usage
TraceOrders

### Parameter
none

### Return Value
**true** Tracing is currently switched on
**false** Tracing is switched off

### Example
```cs
protected override void OnInit()
{
ClearTraceWindow();
TraceOrders = true;
}
```

## Quantity
See [*Position.Quantity*](#positionquantity), [*Position.PositionType*](#positionpositiontype).

## ReplaceOrder()
### Description
Change order, as the name suggests, changes an order.

### Usage
```cs
ReplaceOrder(IOrder iOrder, int quantity, double limitPrice, double stopPrice)
```

### Parameter
|            |                                          |
|------------|------------------------------------------|
| iOrder     | An order object of the type "IOrder"     |
| quantity   | Number of units to be ordered            |
| limitPrice | Limit price. Set this to 0 if not needed |
| stopPrice  | Stop price. Set this to 0 if not needed  |

### Example
```cs
private IOrder stopOrder = null;
protected override void OnCalculate()
{
// If the position is profiting by 10 ticks then set the stop to break-even
if (stopOrder != null
    && Close[0] >= Position.AvgPrice + (10 * TickSize)
        && stopOrder.StopPrice < Position.AvgPrice)
ReplaceOrder(stopOrder, stopOrder.Quantity, stopOrder.LimitPrice, Position.AvgPrice);
}
```
## SetUpProfitTarget()
### Description
Set profit target immediately creates a "take profit" order after an entry order is generated. The order is sent directly to the broker and becomes active immediately.
If the profit target is static, you can also define SetUpProfitTarget() with the OnInit() method.

See [*SetUpStopLoss()*](#setupstoploss), [*SetUpTrailStop()*](#setuptrailstop).

### Usage
```cs
SetUpProfitTarget(double currency)
SetUpProfitTarget(CalculationMode mode, double value)
SetUpProfitTarget(string fromEntry signal, CalculationMode mode, double value)
```

### Parameter
|                  |                                                                                             |
|------------------|---------------------------------------------------------------------------------------------|
| currency         | Sets the profit target in a currency, for example 500€.                                     |
| mode             | Potential values can be: CalculationMode.Percent (display in percent); CalculationMode.Price (display as price value); CalculationMode.Ticks (display in ticks or pips)                                                         |
| value            | The distance between entry price and profit target. This is dependent upon the „mode" but generally refers to a monetary value, a percentage or a value in ticks.                                                                |
| fromEntry signal | The name of the entry signal for which the profit target is to be generated. The amount is taken from the entry order referenced.                                                                                                |

### Example
```cs
protected override void OnInit()
{
// Creates a Target Order 20 ticks above the market
SetUpProfitTarget(CalculationMode.Ticks, 20);
}
```


## SetUpStopLoss()
### Description
Set stop loss creates a stop loss order after an entry order is placed. The order is sent directly to the broker and becomes effective immediately.

If the stop loss is static, then SetUpStopLoss() can be defined with the OnInit() method.

See [*SetUpProfitTarget()*](#setupprofittarget), [*SetUpTrailStop()*](#setuptrailstop).

### Usage
```cs
SetUpStopLoss(double currency)
SetUpStopLoss(double currency, bool simulated)
SetUpStopLoss(CalculationMode mode, double value)
SetUpStopLoss(string fromEntry signal, CalculationMode mode, double value, bool simulated)
```

### Parameter
|                  |                                                                                                  |
|------------------|--------------------------------------------------------------------------------------------------|
| currency         | The difference between the stop loss and the entry price (=risk) in a currency, such as 500€     |
| mode             | Potential values can be: CalculationMode.Percent (display in percent); CalculationMode.Price (display as price value); CalculationMode.Ticks (display in ticks or pips)                                                              |
| simulated        | When set to "true," the stop order does not go live (as a market order) until the price has „touched" it for the first time (meaning that it is executed just as it would be under real market conditions).                            |
| value            | The distance between stop price and profit target. This is dependent upon the „mode" but generally refers to a monetary value, a percentage or a value in ticks.                                                                     |
| fromEntry signal | The name of the entry signal for which the stop order is to be generated. The amount is taken from the entry order referenced.                                                                                                           |

### Example
```cs
protected override void OnInit()
{
// Sets profitTarget 15 Ticks above the market
SetUpStopLoss("MACDEntry", CalculationMode.Ticks, 15, true);
}
```

## SetUpTrailStop()
### Description
Set trail stop creates a trail stop order after an entry order is generated. Its purpose is to protect you from losses, and after reaching break-even, to protect your gains.

The order is sent directly to the broker and becomes effective immediately.

If the stop loss price and the offset value are static, you can define SetUpTrailStop() with the OnInit() method.

If you use SetUpTrailStop() within the [*OnCalculate()*](#oncalculate) method, you must make sure that the parameters are readjusted to the initial value, otherwise the most recently used settings will be used for the new position.

**Functionality:**

Assuming that you have SetUpTrailStop(CalculationMode.Ticks, 30) selected:

In a long position, the stop will be 30 ticks from the previously reached high. If the market makes a new high, the stop will be adjusted. However, the stop will no longer be moved downwards.

In a short position, this behavior starts with the most recent low.

**Tips:**

It is not possible to use SetUpStopLoss and SetUpTrailStop for the same position at the same time within one strategy. The SetUpStopLoss() method will always have precedence over the other methods.

However, it is possible to use both variants parallel to each other in the same strategy if they are referencing different entry signals.

Partial executions of a single order will cause a separate trading stop for each partial position.

If a SetUpProfitTarget() is used in addition to a SetUpTrailStop(), then both orders will be automatically linked to form an OCO order.

It is always a stop market order that is generated, and not a stop limit order.

If a position is closed by a different exit order within the strategy, then the TrailingStopOrder is automatically deleted.

See [*SetUpStopLoss()*](#setupstoploss), [*SetUpProfitTarget()*](#setupprofittarget).

### Usage
```cs
SetUpTrailStop(double currency)
SetUpTrailStop(double currency, bool simulated)
SetUpTrailStop(CalculationMode mode, double value)
SetUpTrailStop(string fromEntry signal, CalculationMode mode, double value, bool simulated)
```

### Parameter
|                  |                                                                                            |
|------------------|--------------------------------------------------------------------------------------------|
| currency         | The distance between the stop loss and the entry price                                     |
| mode             | Possible values are:   CalculationMode.Percent; CalculationMode.Ticks                      |
| simulated        | When set to "true," the stop order does not go live (as a market order) until the price has „touched" it for the first time (meaning that it is executed just as it would be under real market conditions).                      |
| value            | The distance between stop price and profit target. This is dependent upon the „mode" but generally refers to a monetary value, a percentage or a value in ticks.                                                               |
| fromEntry signal | The name of the entry signal for which the stop order is to be generated. The amount is taken from the entry order referenced.                                                                                                     |

### Example
```cs
protected override void OnInit()
{
// Sets a trailing at the low of the last candle
    SetUpTrailStop(CalculationMode.Price, Low[0]);
}
```
## StrategyOrderParameters
## Description
This class aggregates all properties needed to submit the order.

See [*SubmitOrder()*](#submitorder), [*CloseLongTrade()*](#closelongtrade), [*CloseShortTrade()*](#closeshorttrade).

### Usage
```cs
public class StrategyOrderParameters
    {
        public OrderDirection Direction { get; set; }
        public OrderMode Mode { get; set; } = OrderMode.Direct;
        public OrderType Type { get; set; }
        public bool LiveUntilCancelled { get; set; }
        public int Quantity { get; set; }
        public double Price { get; set; }
        public double StopPrice { get; set; }
        public string SignalName { get; set; } = String.Empty;
        public IInstrument Instrument { get; set; }
        public ITimeFrame TimeFrame { get; set; }
        public string FromEntrySignal { get; set; } = String.Empty;
    }
```
### Parameter
|                     |                                                                    |
|---------------------|--------------------------------------------------------------------|
| OrderDirection         | Possible values are: **orderDirection.Buy** (Buy order for a long entry); **orderDirection.Sell** (Sell order for closing a long position)                |
|OrderMode            | One of three possible positions in the market: Direct, Dynamic, Synthetic  |
| OrderType           | Possible values: OrderType.Limit, OrderType.Market, OrderType.Stop, OrderType.StopLimit                                               |
| LiveUntilCancelled  | The order will not be deleted at the end of the bar, but will remain active until removed with [*Order.Cancel*](#ordercancel) or until it reaches its expiry (see [*TimeInForce*](#timeinforce)).         | 
| Quantity            | Amount                                                             |
| Price               | Limit value. Inputting a 0 makes this parameter irrelevant         |
| StopPrice           | Stop value. Inputting a 0 makes this parameter irrelevant          |
| SignalName          | An unambiguous signal name (string)                                |
| Instrument          | The trading instrument in which the position exists.               |
| TimeFrame           | The TimeFrame, which is valid for the order.                       |
| FromEntrySignal     | The name of the attached entry signal                              |


## SubmitOrder()
### Description
Submit order creates a user-defined order. For this order, no stop or limit order is placed in the market. All AgenaTrader control mechanisms are switched off for this order type. The user is responsible for managing the various stop and target orders, including partial executions.

See [*OnOrderChanged()*](#onorderchanged), [*OnOrderExecution()*](#onorderexecution).

### Usage
See [*StrategyOrderParameters()*](#strategyorderparameters)

### Parameter
See [*StrategyOrderParameters()*](#strategyorderparameters)


### Return Value
an order object of the type "IOrder"

### Example
```cs
// Limit Long order
Submit Limit Buy
var order = SubmitOrder(new StrategyOrderParameters
                {
                    Direction = OrderDirection.Buy,
                    Type = OrderType.Limit,
                    Mode = orderMode,
                    Price = limitPrice,
                    Quantity = quantity,
                    SignalName = entryName,
                    Instrument = Instrument,
                    TimeFrame = TimeFrame,
                    LiveUntilCancelled = true
                });

// Short Market order
Submit Sell Market
var order = SubmitOrder(new StrategyOrderParameters
            {
                Direction = OrderDirection.Sell,
                Type = OrderType.Market,
                Mode = ordermode,
                Quantity = quantity,
                SignalName = entryName,
                Instrument = Instrument,
                TimeFrame = TimeFrame
            });
```

## TimeInForce
### Description
The time in force property determines how long an order is valid for. The validity period is dependent upon which values are accepted by a broker.

TimeInForce is specified with the [*OnInit()*](#oninit) method.

Permitted values are:
TimeInForce.day
TimeInForce.loc
TimeInForce.gtc (GTC = good till canceled)
TimeInForce.gtd

**Default:** TimeInForce.GTC

### Usage
**TimeInForce**

### Example
```cs
protected override void OnInit()
{
TimeInForce = TimeInForce.Day;
}
```

## Trade
### Description
Trade is an object containing information about trades that have been executed by a strategy or are currently running.

The individual properties are:

-   **Trade.AvgPrice**
    Average entry price

-   **Trade.ClosedProfitLoss**
    Profit or loss already realized

-   **Trade.Commission**
    Commissions

-   **Trade.CreatedDateTime**
    Time at which the trade was created

-   **Trade.EntryReason**
    Description of the entry signal
    For strategies: signal entry name

-   **Trade.ExitDateTime**
    Time at which the trade was closed

-   **Trade.ExitPrice**
    Exit price

-   **Trade.ExitReason**
    Description of the exit signal
    For strategies: name of the strategy

-   **Trade.Instrument**
    Description of the trading instrument

-   **Trade.PositionType**
    Positioning within the market
    -   PositionType.Flat
    -   PositionType.Long
    -   PositionType.Short

-   **Trade.OpenProfitLoss**
    Unrealized profit/loss of a running position

-   **Trade.ProfitCurrency**
    Profit or loss in the currency that the account is held in

-   **Trade.ProfitLoss**
    Profit or loss

-   **Trade.ProfitPercent**
    Profit or loss in percent

-   **Trade.ProfitPercentWithCommission**
    Profit or loss in percent with commissions

-   **Trade.ProfitPoints**
    Profit or loss in points/pips

-   **Trade.Quantity**
    Quantity of stocks/contracts/ETFs/etc.

-   **Trade.TimeFrame**
    Timeframe in which the trade was opened

-   **Trade.Url**
    URL for the snapshot of the chart at the moment of creation

### Example
```cs
protected override void OnDispose()
{
  if (Performance.AllTrades.Count < 1) return;
  foreach (ITrade trade in Performance.AllTrades)
  {
    Print("Trade #"+trade.Id);
    Print("--------------------------------------------");
    Print("Average price " + trade.AvgPrice);
    Print("Realized P/L " + trade.ClosedProfitLoss);
    Print("Commissions " + trade.Commission);
    Print("Time of entry " + trade.CreatedDateTime);
    Print("Entry reason " + trade.EntryReason);
    Print("Time of exit " + trade.ExitDateTime);
    Print("Exit price " + trade.ExitPrice);
    Print("Exit reason " + trade.ExitReason);
    Print("Instrument " + trade.Instrument);
    Print("Positioning " + trade.PositionType);
    Print("Unrealized P/L " + trade.OpenProfitLoss);
    Print("P/L (currency) " + trade.ProfitCurrency);
    Print("P/L " + trade.ProfitLoss);
    Print("P/L (in percent) " + trade.ProfitPercent);
    Print("P/L (% with commission)" + trade.ProfitPercentWithCommission);
    Print("PL (in points) " + trade.ProfitPoints);
    Print("Quantity " + trade.Quantity);
    Print("Timeframe " + trade.TimeFrame);
    Print("URL for the snapshot " + trade.Url);
  }
}
```

## Unmanaged
# Backtesting and Optimization

## Performance Characteristics
Performance characteristics are the various factors that can be calculated for a list of trades. The trades can be generated by a strategy in real-time or based on a backtest.

The following are available:

-   all trades
-   all long trades
-   all short trades
-   all winning trades
-   all losing trades

See [*Performance*](#performance).

The individual factors are:

**AvgEtd**
The average drawdown at the end of a trade
&lt;TradeCollection&gt;.TradesPerformance.&lt;TradesPerformanceValues&gt;.AvgEtd

```cs
Print("Average ETD of all trades is: " + Performance.AllTrades.TradesPerformance.Currency.AvgEtd);
```

**AvgMae**
Average maximum adverse excursion
&lt;TradeCollection&gt;.TradesPerformance.&lt;TradesPerformanceValues&gt;.AvgMae

```cs
Print("Average MAE of all trades is: " + Performance.AllTrades.TradesPerformance.Currency.AvgMae);
```

**AvgMfe**
Average maximum favorable excursion
&lt;TradeCollection&gt;.TradesPerformance.&lt;TradesPerformanceValues&gt;.AvgMfe

```cs
Print("Average MFE of all trades is: " + Performance.AllTrades.TradesPerformance.Currency.AvgMfe);
```

**AvgProfit**
Average profit for all trades
&lt;TradeCollection&gt;.TradesPerformance.&lt;TradesPerformanceValues&gt;.AvgProfit

```cs
Print("Average profit of all trades is: " + Performance.AllTrades.TradesPerformance.Currency.AvgProfit);
```

**CumProfit**
The cumulative winnings over all trades
&lt;TradeCollection&gt;.TradesPerformance.&lt;TradesPerformanceValues&gt;.CumProfit

```cs
Print("Average cumulative profit of all trades is: " + Performance.AllTrades.TradesPerformance.Currency.CumProfit);
```

**DrawDown**
The drawdown for all trades
&lt;TradeCollection&gt;.TradesPerformance.&lt;TradesPerformanceValues&gt;.DrawDow

```cs
Print("Drawdown of all trades is: " + Performance.AllTrades.TradesPerformance.Currency.DrawDown);
```

**LargestLoser**
The largest losing trade
&lt;TradeCollection&gt;.TradesPerformance.&lt;TradesPerformanceValues&gt;.LargestLoser

```cs
Print("Largest loss of all trades is: " + Performance.AllTrades.TradesPerformance.Currency.LargestLoser);
```

**LargestWinner**
The largest winning trade
&lt;TradeCollection&gt;.TradesPerformance.&lt;TradesPerformanceValues&gt;.LargestWinner

```cs
Print("Largest win of all trades is: " + Performance.AllTrades.TradesPerformance.Currency.LargestWinner);
```

**ProfitPerMonth**
The total performance (wins/losses) for the month (also in percent)
&lt;TradeCollection&gt;.TradesPerformance.&lt;TradesPerformanceValues&gt;.ProfitPerMonth

```cs
Print("Profit per month of all trades is: " + Performance.AllTrades.TradesPerformance.Currency.ProfitPerMonth);
```

**StdDev**
    The standard deviation for the wins/losses. With this, you are able to identify outliers. The smaller the standard deviation, the higher the expectation of winnings.

**All factors are double values.**

![Performance Characteristics](./media/image10.png)
