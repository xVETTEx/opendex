# BOLD #5: Matching Protocol

## Overview
Node can choose which matcher match its orders. Anyone can be matcher. Matcher runs orderbook and find matches, when match is found, matcher sends `SwapRequest` message to both party of trade. Orderbook has delay time, which define how much time peer have to accept or decline swap request. Delay is expressed in seconds and delay of local orderbook is 0.
Node can give its order to matcher, matcher of that one orderbok can again place order to orderbook with higher delay. And so on...
When Matcher create SwapRequest, matcher send it back to slower delay orderbook until owner of order get it, then owner create response message(SwapAccepter or SwapFailed) and send it to first matcher, which send it to matcher with higher delay until matcher which sended SwapRequest get response.
Matcher can choose not to take orders from some nodes. As an example if swap never or very rarely succeed with some node, matcher can stop taking orders from this node. 

### Matching rules
If there are in orderbook many makers in same price, matcher should match oldest order first.

Matcher shouldn't match orders where each party is in ban list of other party.

Market orders, should be matched with current best price order.

Limit orders have to be matched with right price.

Post only order shouldn't be placed to orderbook or matched if it could be matched immediatly.

Orders with trigger should be placed orderbook when trigger price is reached, not earlier.

## Overview
Node can subscribe all events on matcher's orderbook, so it can be sure that matcher follow matching rules. Matcher is not required to accept any subscribe reguests, but no one may want to use orderbook that can't be verified of following rules.
Node will broadcast messages below, which applies to orders in this orderbook to everyone who have subscribed orderbook. 
1. Order Message (0x06)
2. OrderInvalidation Message (0x07)
3. SwapRequest Message (0x0c)
4. SwapAccepted Message (0x0d)
5. SwapComplete Message (0x0e)
6. SwapFailed Message (0x0f)
7. NodestateUpdate Message(0x02), only changes in banlist of nodes which have used orderbook in last 24 hours. If node that haven't been trading in last 24 hours place order, its banlist should be broadcasted to subscribers.

When peer start subscribing orderbook next things will be broadcasted:
1. All current orders of orderbook
2. All banlists of nodes that have used orderbook in last 24 hours.
3. Last swap protocol messages of orders that have swap in progress.

### Subscribe Message (0x)
	`string orderbook_id = 1`
	Orderbook which you want subscribe.
	`state = 2`
	If state is 'subscribe', node want start subscribing. If state is 'unsubscribe', node stop subscribing.

### SubscribeResponse Message(0x)
	`orderbook id = 1`
	`state = 2`
	If state is 'accepted', request to subscribe is accepted. If state is 'unaccepted', request to subscribe is unaccepted, or 		request to unsubscribe is recieved.
