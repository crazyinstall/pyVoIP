SIP - Session Initiation Protocol
##################################

The SIP module receives, parses, and responds to all incoming SIP requests/messages.  If appropriate, it then forwards them to the *callback* method of :ref:`VoIPPhone`.

Errors
*******

There are two errors under ``pyVoIP.SIP``.

.. _InvalidAccountInfoError:

*exception* SIP.\ **InvalidAccountInfoError**
  This is thrown when :ref:`SIPClient` gets a bad response when trying to register with the PBX/VoIP server.  This error also kills the SIP REGISTER thread, so you will need to call SIPClient.stop() then SIPClient.start().

.. _sip-parse-error:

*exception* SIP.\ **SIPParseError**
  This is thrown when :ref:`SIPMessage` is unable to parse a SIP message/request.

.. _Enums:

Enums
******

SIP.\ **SIPMessageType**
  SIPMessageType is an IntEnum with two attributes.  It's stored in ``SIPMessage.type`` to effectivly parse the message.
  
  SIPMessageType.\ **MESSAGE**
    This SIPMessageType is used to signify the message was a SIP request.
    
  SIPMessageType.\ **RESPONSE**
    This SIPMessageType is used to signify the message was a SIP response.
    
SIP.\ **SIPStatus**
  SIPStatus is used for :ref:`SIPMessage`'s with SIPMessageType.RESPONSE.  They will not all be listed here, but a complete list can be found on `Wikipedia <https://en.wikipedia.org/wiki/List_of_SIP_response_codes>`_.  SIPStatus has the following attributes:
  
    status.\ **value**
      This is the integer value of the status.  For example, ``SIPStatus.OK.value`` is equal to ``int(200)``.
      
    status.\ **phrase**
      This is the string value of the status, usually written next to the number in a SIP response. For example, ``SIPStatus.TRYING.phrase`` is equal to ``'Trying'``.
      
    status.\ **description**
      This is the string value of the description of the status, it can be useful for debugging.  For example, ``SIPStatus.OK.description`` is equal to ``'Request successful'``  Not all responses have a description.
  
  Here are a few common SIPStatus' and their attributes in the order of value, phrase, description:
  
  SIPStatus.\ **TRYING**
    100, 'Trying', 'Extended search being performed, may take a significant time'
    
  SIPStatus.\ **RINGING**
    180, 'Ringing', 'Destination user agent received INVITE, and is alerting user of call'
  
  SIPStatus.\ **OK**
    200, 'OK', 'Request successful'
    
  SIPStatus.\ **BUSY_HERE**
    486, 'Busy Here', 'Callee is busy'

Classes
********

.. _SIPClient:

SIPClient
==========

The SIPClient class is used to communicate with the PBX/VoIP server.  It is responsible for registering with the server, and receiving phone calls.

*class* SIP.\ **SIPClient**\ (server, port, username, password, myIP=None, myPort=5060, callCallback=None)
    The *server* argument is your PBX/VoIP server’s IP, represented as a string.
    
    The *port* argument is your PBX/VoIP server’s port, represented as an integer.

    The *username* argument is your SIP account username on the PBX/VoIP server, represented as a string.

    The *password* argument is your SIP account password on the PBX/VoIP server, represented as a string.
    
    The *myIP* argument is used to receive incoming SIP requests and responses. If left as None, the SIPClient will bind to 0.0.0.0.

    The *myPort* argument is the port SIPClient will bind to, to receive incoming SIP requests and responses. The default for this protocol is port 5060, but any port can be used.

    The *callCallback* argument is the callback function for :ref:`VoIPPhone`.  VoIPPhone will process the SIP request, and perform the appropriate actions.

  **recv**\ ()
    This method is called by SIPClient.start() and is responsible for receiving and parsing through SIP requests.  **This should not be called by the** :term:`user`.
    
  **start**\ ()
    This method is called by :ref:`VoIPPhone`.start().  It starts the REGISTER and recv() threads.  It is also what initiates the bound port.  **This should not be called by the** :term:`user`.
    
  **stop**\ ()
    This method is called by :ref:`VoIPPhone`.stop(). It stops the REGISTER and recv() threads.  It will also close the bound port.  **This should not be called by the** :term:`user`.
    
  **genCallID**\ ()
    This method is called by other 'gen' methods when a new Call-ID header is needed.  See `RFC 3261 Section 20.8 <https://tools.ietf.org/html/rfc3261#section-20.8>`_.  **This should not be called by the** :term:`user`.
    
  **getSIPVersoinNotSupported**\ ()
    This method is called by the recv() thread when it has received a SIP message that is not SIP version 2.0.
    
  **genAuthorization**\ (request):
    This calculates the authroization hash in response to the WWW-Authenticate header.  See `RFC 3261 Section 20.7 <https://tools.ietf.org/html/rfc3261#section-20.7>`_.  The *request* argument should be a 401 Unauthorized response.  **This should not be called by the** :term:`user`.
    
  **genRegister**\ (request)
    This method generates a SIP REGISTER request. The *request* argument should be a 401 Unauthorized response.   **This should not be called by the** :term:`user`.
    
  **genBusy**\ (request)
    This method generates a SIP 486 'Busy Here' response.  The *request* argument should be a SIP INVITE request.
    
  **genRinging**\ (request)
    This method generates a SIP 180 'Ringing' response.  The *request* argument should be a SIP INVITE request.
    
  **genAnswer**\ (request, sess_id, ms, sendtype)
    This method generates a SIP 200 'OK' response.  Which, when in reply to an INVITE request, tells the server the :term:`user` has answered.  **This should not be called by the** :term:`user`.
    
    The *request* argument should be a SIP INVITE request.
    
    The *sess_id* argument should be a string casted integer.  This will be used for the SDP o tag.  See `RFC 4566 Section 5.2 <https://tools.ietf.org/html/rfc4566#section-5.2>`_.  The *sess_id* argument will also server as the *<sess-version>* argument in the SDP o tag.
    
    The *ms* argument should be a list of parsed SDP m tags, found in the :ref:`SIPMessage`.body attribute.  This is used to generate the response SDP m tags.   See `RFC 4566 Section 5.14 <https://tools.ietf.org/html/rfc4566#section-5.14>`_.
    
    The *sendtype* argument should be a RTP.\ :ref:`TransmitType<transmittype>` enum.  This will be used to generate the SDP a tag.   See `RFC 4566 Section 6 <https://tools.ietf.org/html/rfc4566#section-6>`_.
    
  **genBye**\ (request)
    This method generates a SIP BYE request.  This is used to end a call. The *request* argument should be a SIP INVITE request.  **This should not be called by the** :term:`user`.
    
  **bye**\ (request)
    This method is called by :ref:`VoIPCall`.hangup().  It calls genBye(), and then transmits the generated request.  **This should not be called by the** :term:`user`.
    
  **deregister**\ ()
    This method is called by SIPClient.stop() after the REGISTER thread is stopped.  It will generate and transmit a REGISTER request with an Expiration of zero.  Telling the PBX/VoIP server it is turning off.  **This should not be called by the** :term:`user`.
    
  **register**\ ()
    This method is called by the REGISTER thread.  It will generate and transmit a REGISTER request telling the PBX/VoIP server that it will be online for at least 300 seconds.  The REGISTER thread will call this function every 295 seconds.  **This should not be called by the** :term:`user`.
    
.. _SIPMessage:

SIPMessage
==========

The SIPMessage class is used to parse SIP requests and responses and makes them easily processed by other classes.

*class* SIP.\ **SIPMessage**\ (data)
    The *data* argument is the SIP message in bytes.  It is then passed to SIPMessage.parse().
  
  SIPMessage has the following attributes:
  
    SIPMessage.\ **heading**
      This attribute is the first line of the SIP message as a string.  It contains the SIP Version, and the method/response code.
      
    SIPMessage.\ **type**
      This attribute will be a :ref:`SIPMessageType<enums>`.
      
    SIPMessage.\ **status**
      This attribute will be a :ref:`SIPStatus<enums>`.  It will be set to ``int(0)`` if the message is a request.
      
    SIPMessage.\ **method**
      This attribute will be a string representation of the method.  It will be set to None if the message is a response.
      
    SIPMessage.\ **headers**
      This attribute is a dictionary of all the headers in the request, and their parsed values.
      
    SIPMessage.\ **body**
      This attribute is a dictionary of all the SDP tags in the request, and their parsed values.
      
    SIPMessage.\ **authentication**
      This attribute is a dictionary of a parsed Authentication header.  There are two authentication headers: Authorization, and WWW-Authenticate.  See RFC 3261 Sections `20.7 <https://tools.ietf.org/html/rfc3261#section-20.7>`_ and `20.44 <https://tools.ietf.org/html/rfc3261#section-20.44>`_ respectively.
      
    SIPMessage.\ **raw**
      This attribute is an unparsed version of the *data* argument, in bytes.
      
  **summary**\ ()
    This method returns a string representation of the SIP request.
    
  **parse**\ (data)
    This method is called by the initialization of the class.  It decides the SIPMessageType, and sends it to the corresponding parse function.  *Data* is the original *data* argument in the initialization of the class.  **This should not be called by the** :term:`user`.

  **parseSIPResponse**\ (data)
    This method is called by parse().  It sets the *header*, *version*, and *status* attributes and may raise a :ref:`SIPParseError<sip-parse-error>` if the SIP response is an unsupported SIP version.  It then calls parseHeader() for each header in the request. *Data* is the original *data* argument in the initialization of the class.  **This should not be called by the** :term:`user`.
    
  **parseSIPMessage**\ (data)
    This method is called by parse().  It sets the *header*, *version*, and *method* attributes and may raise a :ref:`SIPParseError<sip-parse-error>` if the SIP request is an unsupported SIP version.  It then calls parseHeader() and parseBody() for each header or tag in the request respectively. *Data* is the original *data* argument in the initialization of the class.  **This should not be called by the** :term:`user`.
    
  **parseHeader**\ (header, data)
    This method is called by parseSIPResponse() and parseSIPMessage().  The *header* argument is the name of the header, i.e. 'Call-ID' or 'CSeq', represented as a string.  The *data* argument is the value of the header, i.e. 'Ogq-T7iBmNozoUu3GL9Lvg..' or '1 INVITE', represented as a string.  **This should not be called by the** :term:`user`.
    
  **parseBody**\ (header, data)
    This method is called by parseSIPResponse() and parseSIPMessage().  The *header* argument is the name of the SDP tag, i.e. 'm' or 'a', represented as a string.  The *data* argument is the value of the header, i.e. 'audio 56704 RTP/AVP 0' or 'sendrecv', represented as a string.  **This should not be called by the** :term:`user`.
