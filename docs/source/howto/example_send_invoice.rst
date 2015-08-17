Как отправить счет-фактуру
==========================

Рассмотрим последовательность действий к функциям интеграторского интерфейса Диадока, которые требуется совершить при отправке счета-фактуры (СФ), корректировочного счета-фактуры (КСФ), исправления счета-фактуры (ИСФ).

1. Формирование счета-фактуры
-----------------------------

Если на стороне интеграционного решения не предусмотрено функциональности для формирования XML-документов, соответствущих утвержденным форматам, то первым делом, продавец должен сгенерировать СФ/ИСФ/КСФ/ИКСФ, используя команду :doc:`../http/GenerateInvoiceXml`.

	-  если нужно сгенерировать СФ, то значение GET-параметра ``invoiceType`` должно принимать значение ``Invoice``,
	
	-  если нужно сгенерировать КСФ, то значение GET-параметра ``invoiceType`` должно принимать значение ``InvoiceCorrection``,
	
	-  если нужно сгенерировать ИСФ, то значение GET-параметра ``invoiceType`` должно принимать значение ``InvoiceRevision``,
	
	-  если нужно сгенерировать ИКСФ, то значение GET-параметра ``invoiceType`` должно принимать значение ``InvoiceCorrectionRevision``,
	   
В теле запроса должны содержаться данные для изготовления СФ/ИСФ/КСФ/ИКСФ:
	
	-  в виде сериализованной структуры :doc:`../proto/InvoiceInfo` для типов документов ``Invoice`` или ``InvoiceRevision``
	
	-  в виде сериализованной структуры :doc:`../proto/InvoiceCorrectionInfo` для типов документов ``InvoiceCorrection`` или ``InvoiceCorrectionRevision``.
	   
Например HTTP-запрос для генерации СФ выглядит следующим образом:

::

    POST https://diadoc-api.kontur.ru/GenerateInvoiceXml?invoiceType=Invoice HTTP/1.1
    Host: diadoc-api.kontur.ru
    Authorization: DiadocAuth ddauth_api_client_id=testClient-8ee1638deae84c86b8e2069955c2825a
    Content-Length: 1252
    Connection: Keep-Alive

    <Сериализованная структура InvoiceInfo>

В теле ответа содержится XML-файл СФ/ИСФ/КСФ/ИКСФ, построенный на основании данных из запроса.

Успешный ответ сервера выглядит так:
::

    HTTP/1.1 200 OK
    Content-Length: 598

    <XML-файл СФ>

Файл СФ/ИСФ изготавливается в соответствии с `XML-схемой <https://diadoc.kontur.ru/sdk/xsd/ON_SFAKT_1_897_01_05_01_02.xsd>`__, которой должны удовлетворять XML-счета-фактуры, согласно приказу ФНС.

Файл КСФ/ИКСФ изготавливается в соответствии с другой утвержденной ФНС `XML-схемой <https://diadoc.kontur.ru/sdk/xsd/ON_KORSFAKT_1_911_01_05_01_02.xsd>`__. 

Имя файла СФ/ИСФ/КСФ/ИКСФ (формат которого также определяет приказ ФНС) возвращается в стандартном HTTP-заголовке ``Content-Disposition``.

2. Отправка счета-фактуры
-------------------------

После того, как у вас есть XML-файл СФ/ИСФ/КСФ/ИКСФ, его нужно отправить с помощью команды :doc:`../http/PostMessage`. 

Для этого нужно подготовить структуру :doc:`../proto/MessageToPost` следующим образом:

-  в значение атрибута *FromBoxId* указываем идентификатор ящика отправителя,

-  в значение атрибута *ToBoxId* указываем идентификатор ящика получателя

-  для передачи XML-файла СФ/ИСФ/КСФ/ИКСФ нужно использовать атрибут *Invoices*, описываемый структурой *XmlDocumentAttachment*

	-  внутри структуры *XmlDocumentAttachment* находится вложенная структура *SignedContent*,
	
	-  сам XML-файла нужна передать в атрибут *Content*, подпись продавца в атрибут *Signature*
	   
Описание структур, используемых при отправке СФ:

.. code-block:: protobuf

    message MessageToPost {
        required string FromBoxId = 1;
        optional string ToBoxId = 2;
        repeated XmlDocumentAttachment Invoices = 3;
    }

    message XmlDocumentAttachment {
        required SignedContent SignedContent = 1;
        optional string Comment = 3;
    }

    message SignedContent {
        optional bytes Content = 1;
        optional bytes Signature = 2;
    }

3. Формирование извещения покупателя
------------------------------------