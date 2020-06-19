
====
.. index:: ! Ahri

.. _Ahri:
Ahri
====

``Ahri`` is the SDK for building Kumiho apps.

.. code-block:: javascript

    const Ahri = window.Ahri;

------------------------------------------------------------------------------


compileSolidity
---------------------
.. code-block:: javascript

    async compileSolidity(solidity, { version, host })
    
Compiles Solidity. A request is sent to ``${host}/api/compile``. The 'host' parameter defaults to ``https://kumiho.org``. The 'version' parameter defaults to ``v0.5.6+commit.b259423e``. The result is returned as ``Solidity.sol``.

.. code-block:: javascript

    const result = await Ahri.compileSolidity(solidity);
    
    console.log(result);
    
    //{
    //  contracts: {Solidity.sol: {…}},
    //  sources: {Solidity.sol: {…}},
    //}

------------------------------------------------------------------------------


deployContract
---------------------
.. code-block:: javascript

    async deployContract(abi, bin, args, options = {})
    
Deploys a Contract.

.. code-block:: javascript

    const contract = await Ahri.compileSolidity(solidity);
    const className = Object.keys(contract.contracts['Solidity.sol'])[0];
    const abi = contract.contracts['Solidity.sol'][className].abi;
    const bin = '0x' + contract.contracts['Solidity.sol'][className].evm.bytecode.object;
    
    const depResult = await Ahri.deployContract(abi, bin, [1, 'test'], options = {gas: 1500000})

------------------------------------------------------------------------------


callByUrl
---------------------
.. code-block:: javascript

    async callByUrl(url, method, args, value = 0, options = {})
    
``callByUrl`` invokes the Smart Contract mapped by the 'url'. Note that nothing will be returned for a ``POST`` request.

----------
Parameters
----------

1. ``url`` - ``String``: The URL by which the target Contract is mapped.
2. ``method`` - ``'GET'|'POST'``: The method with which the target Contract is called. Use ``GET`` for pure functions, and ``POST`` for payable functions or any other functions.
3. ``args`` - ``Array``: The arguments to be passed to the Contract.
4. ``value`` - ``int``: The amount of value to be transferred (in peb). Ignored when ``method === 'GET'``. (default: 0)
5. ``options``

----------
Options
----------

1. ``gas`` - ``int``: The upper limit for gas. (default: 1500000)
2. ``gasPrice`` - ``int``: The gas price. (default: caver default)
3. ``anonymous`` - ``Boolean``: (v1.0.1) When ``anonymous === true`` and ``method === 'GET'``, ``msg.sender`` will be set to ``0x0000000000000000000000000000000000000000``, so that no access request to Kaikas occurs.

----------
Returns
----------

1. ``Object`` - The value returned from the Contract.

.. code-block:: javascript

    let result = await Ahri.callByUrl('demo.klay/get', 'GET', []);
    console.log(result);
    //{0: "1", __length__: 1}
    
    await Ahri.callByUrl('demo.klay/set', 'POST', [3]);
    
    result = await Ahri.callByUrl('demo.klay/get', 'GET', []);
    console.log(result);
    //{0: "3", __length__: 1}

------------------------------------------------------------------------------


callByAddress
---------------------
.. code-block:: javascript

    async callByAddress(address, method, functionName, args, argTypes, resultTypes, value = 0, options = {})

``callByAddress`` requests to invoke a Smart Contract with its address. Note that nothing will be returned for ``POST`` requests.

----------
Parameters
----------

1. ``address`` - ``Address``: Address of the target Contract
2. ``method`` - ``'GET'|'POST'``: The method with which the target Contract is called. Use ``GET`` for pure functions, and ``POST`` for payable functions or any other functions.
3. ``functionName`` - ``string``: The name of the function to be called.
4. ``args`` - ``Array``: The arguments to be passed to the Contract.
5. ``argTypes`` - ``Array<Type>``: The types for the given arguments: ``['uint256', 'string', 'address']``
6. ``resultTypes`` - ``Array<Type>``: The types for the return values: ``['uint256', 'string', 'address']``
7. ``value`` - ``int``: The amount of value to be transferred (in peb). Ignored when ``method === 'GET'``. (default: 0)
8. ``options``

----------
Options
----------

1. ``gas`` - ``int``: The upper limit for gas. (default: 1500000)
2. ``gasPrice`` - ``int``: The gas price. (default: caver default)
3. ``anonymous`` - ``Boolean``: (v1.0.1) When ``anonymous === true`` and ``method === 'GET'``, ``msg.sender`` will be set to ``0x0000000000000000000000000000000000000000``, so that no access request to Kaikas occurs.

.. code-block:: javascript

    let result = await Ahri.callByAddress('0x56f8eF1d4986460df8783d2A2147C1069203F848', 'GET', 'getValue', [], [], ['uint256']);
    console.log(result);
    //{0: "1", __length__: 1}
    
    await Ahri.callByAddress('0x56f8eF1d4986460df8783d2A2147C1069203F848', 'POST', 'setValue', [3], ['uint256'], []);
    
    result = await Ahri.callByAddress('0x56f8eF1d4986460df8783d2A2147C1069203F848', 'GET', 'getValue', [], [], ['uint256']);
    console.log(result);
    //{0: "3", __length__: 1}

------------------------------------------------------------------------------


fetch
---------------------
.. code-block:: javascript

    async fetch(url, req = {})

Send a fetch request with the HTTP Header ``Klay-Address`` set to the wallet address.

.. code-block:: javascript

    async fetch(url, req = {}) {
        const response = await fetch(url, {
            ...req,
            headers: {
                ...(req.headers || {}),
                'Klay-Address': caver.klay.accounts.wallet.getAccount(0).address,
            }
        });
        return response;
    }
    
------------------------------------------------------------------------------


getApiInterface
---------------------
.. code-block:: javascript

    async getApiInterface(url)
    
Returns the interface of the Smart Contract mapped by the URL.


.. code-block:: javascript

    const interface = await Ahri.getApiInterface('demo.klay/get');
    console.log(interface);
    
    //{
    //    "address": "0x56f8eF1d4986460df8783d2A2147C1069203F848",
    //    "functionName": "getValue",
    //    "parameters": [],
    //    "result": [
    //        "uint256"
    //    ]
    //}
 
------------------------------------------------------------------------------


getEvents
---------------------
.. code-block:: javascript

    async getEvents(url, fromBlock, toBlock)

Returns the events from the URL.

.. code-block:: javascript

    const events = await Ahri.getEvents('kumiho.klay/redistribution/event/Vote', prevBlockNum + 1, currentBlockNum)

    //[
    //    {
    //        "data": [
    //            "1",
    //            "0x242BCe82d002DB2De7cE19D5b005E7d534C05321"
    //        ],
    //        "blockNumber": 23334472,
    //        "logIndex": 0
    //    },
    //    {
    //        "data": [
    //            "2",
    //            "0x242BCe82d002DB2De7cE19D5b005E7d534C05321"
    //        ],
    //       "blockNumber": 23334480,
    //        "logIndex": 0
    //    },
    //    {
    //        "data": [
    //            "11",
    //            "0x242BCe82d002DB2De7cE19D5b005E7d534C05321"
    //        ],
    //        "blockNumber": 23334508,
    //        "logIndex": 0
    //    }
    //]

------------------------------------------------------------------------------


getRecentEvents
---------------------
.. code-block:: javascript

    async getRecentEvents(url, blockCount)

Returns the events from the URL. Convenience function for ``getEvents(url, [currentBlockNumber] - blockCount, [currentBlockNumber])``

.. code-block:: javascript

    const events = await Ahri.getRecentEvents('kumiho.klay/redistribution/event/Vote', 100)

    //[
    //    {
    //        "data": [
    //            "1",
    //            "0x242BCe82d002DB2De7cE19D5b005E7d534C05321"
    //        ],
    //        "blockNumber": 23334472,
    //        "logIndex": 0
    //    },
    //    {
    //        "data": [
    //            "2",
    //            "0x242BCe82d002DB2De7cE19D5b005E7d534C05321"
    //        ],
    //       "blockNumber": 23334480,
    //        "logIndex": 0
    //    },
    //    {
    //        "data": [
    //            "11",
    //            "0x242BCe82d002DB2De7cE19D5b005E7d534C05321"
    //        ],
    //        "blockNumber": 23334508,
    //        "logIndex": 0
    //    }
    //]

