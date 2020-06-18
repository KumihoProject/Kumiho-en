
========
Kumiho
========

``Kumiho`` is a decentralized web application platform based on Klaytn.
Kumiho supports Chrome and Firefox. You need to install one of the following browser extensions to use it.

- Chrome_
- Firefox_

.. _Chrome: https://chrome.google.com/webstore/category/extensions
.. _Firefox: https://addons.mozilla.org/ko/firefox/extensions/

.. note::
    Kumiho needs :ref:`Foxfire<Foxfire>` and Kaikas pre-installed.

If you want to deploy a new web application, register a domain at http://kumiho.klay/enroll_domain.

------------------------------------------------------------------------------

Deploy Web Application
=============
1. Purchase a domain you want at http://kumiho.klay/enroll_domain. Subdomains are not currently supported. Domains must end in .klay.
2. After you purchase, you can manage the registered domains at http://kumiho.klay/dashboard with your Kaikas account.
3. You can register a web page by clicking + Add Web Page on the domain management page. You can also register a solidity function on Path by clicking
 + Add API on the same page.

------------------------------------------------------------------------------

Web Page
=============
Accessibility is the biggest hurdle to the general adoption of blockchain technology. In fact, many blockchain networks do exist today, but they are usually
isolated and not very accessible to use.

On the other hand, the web is a very accessible interface. Many computers are connected to the internet and are mostly equipped with a web browser.

Simply putting HTML codes to blockchain network does not make it accessible with the usual methods. That is why we provide :ref:`Foxfire<Foxfire>` to do
precisely this.

HTML codes are uploaded to the Klaytn network in the form of a reversed linked list. When a user with Foxfire connects to .klay domain, Kumiho's backend
server :ref:`Ninetales<Ninetales>` fetches the Webloader which loads a web page from the blockchain network. The Webloader then looks at the domain the user is
currently on and get the starting address of the smart contract that holds the HTML codes from Ninetales.

.. code-block:: solidity

    pragma solidity >=0.4.25 <0.6.0;

    contract CodeFrag {
        address prevContract;
        string codeFrag;
        string format;
        string version;
        constructor (address _prevContract, string memory _codeFrag, string memory _format) public {
            prevContract = _prevContract;
            codeFrag = _codeFrag;
            format = _format;
            version = "0.0.1";
        }

        function getVersion () public view returns(string memory) {
            return version;
        }

        function getCodeFragment () public view returns(address, string memory, string memory) {
            return (prevContract, codeFrag, format);
        }
    }

When all the HTML codes are loaded, the Webloader assembles the DOM and show it on the current page.

.. warning::
    ``Foxfire``, ``Ninetales`` themselves do not execute the code from the contract. The Webloader that Foxfire fetches from Ninetales execute the actual code
    on the actual machine. The following code does not work.

    .. code-block:: HTML
    
        <script src="http://kumiho.klay/script.js">
        <style href="http://kumiho.klay/style.css">

    .. code-block:: javascript
    
        fetch('http://kumiho.klay/');
    
    Therefore, you need to bundle your application using a tool such as Webpack. In order to avoid the app getting too big in size, it is recommended to import
    libraries and images from CDNs.

    Please refer to Kumiho's sample application Redistribution_ code.

    .. _Redistribution: https://github.com/KumihoProject/Redistribution

.. note::
    Due to the above reasons, we recommend to write your application as a Single Page Application. But in that case, even a small change in the application
    requires the entire app to be re-uploaded. You can avoid this by splitting your app on different paths.
    
    For example, upload your app on ``/`` and ``/dashboard`` as separate apps.

--------------------------------------------------------------------------

Api
=============
Api is the critical element to implement a Serverless Web Application on Kumiho platform. Web pages can be served from CDNs, however, generally a web
application needs a web server to handle users' requests for security, input validation and database.

EVM itself is a computing platform capable of arithmetic operations, but it is still difficult for EVM to interact with the real web. Caver can directly
interact with smart contracts, but then a version update becomes a daunting task. Kumiho provides an interface similar to conventional REST APIs that are
easy to manage and use.

When you request an API call through :ref:`Ahri<Ahri>` SDK, Ahri brings the address of a smart contract that holds the actual API interface and assemble it
with the requested smart contract function data.

.. code-block:: solidity

    pragma solidity >=0.4.25 <0.6.0;

    contract KumihoInterface {
        address contractAddress;
        string functionName;
        string parameters;
        string result;
        constructor (address _contractAddress, string memory _functionName, string memory _parameters, string memory _result) public {
            contractAddress = _contractAddress;
            functionName = _functionName;
            parameters = _parameters;
            result = _result;
        }
        function getFunctionMeta () public view returns(address, string memory, string memory, string memory) {
            return (contractAddress, functionName, parameters, result);
        }
    }

This code compiles the requested function's ABI, call the function and return the result.