# Mirakl Account configuration

Before starting the installation, please read all instructions and make sure you've gone through the [Prerequisites and recommendations](https://developer.hipay.com/doc/hipay-marketplace-cashout-mirakl-integration/#prerequisites-and-recommendations) section.
 
## Preamble

In order for us to validate the HiPay accounts, the connector needs to upload KYC documents allowing us to identify your merchants.

Thus, you need to configure your Mirakl back office in order to be able to upload these documents for each of your shops. Then, these documents will be retrieved by the HiPay Marketplace cash-out integration for Mirakl and sent to the HiPay platform.

## Configuration

Go in the **Manage Document Types** section by following these steps.

- Log in to your Mirakl operator account.
- In the **Settings** tab, click on **Stores** (in the "Advanced parameters" column).
- Click on **Documents**.

You need to provide the following list of document types by clicking on *Add a document type*:

| Types of professional | Label | Description | Code | 
|-------|-------|-------|------|
| **All professionals** | Bank | Bank account details (“RIB”/IBAN) / Account statement /… | `ALL_PROOF_OF_BANK_ACCOUNT` |
| **Corporation** | Identity card | Copy of a valid identification document of the legal representative | `LEGAL_IDENTITY_OF_REPRESENTATIVE` | 
| **Corporation** || Company Registration | Document certifying company registration issued within the last three months (Kbis extract) | `LEGAL_PROOF_OF_REGISTRATION_NUMBER` | 
| **Corporation** | Distribution of power | Signed Articles of Association with the division of powers | `LEGAL_ARTICLES_DISTR_OF_POWERS` |  
| **Persons** | ID proof | Copy of a valid identification document of the legal representative | `SOLE_MAN_BUS_IDENTITY` |  
| **Persons** | Company Registration  | Document certifying registration issued within the last three months (Kbis extract) | `SOLE_MAN_BUS_PROOF_OF_REG_NUMBER` |  
| **Persons** | Tax status | Document certifying tax status (“auto-entrepreneur” / independent /…) | `SOLE_MAN_BUS_PROOF_OF_TAX_STATUS` |  

You can set whatever you want in the description field. In any case, **the codes must be exactly as displayed in the table**.

Then, you will need to upload the documents on each of the shop pages.

Note: when uploading your files (on a shop page), **only upload files corresponding to the type of your shop**. For example, if the shop is managed by a person, you can only upload the documents that are required "only for persons" as well as the bank account details, but not the documents flagged as being "only for corporations".

If your marketplace only has corporations as shops and no shops held by persons (most probable scenario), **you should not add the document types flagged as being "only for persons" because you would never need to upload such types of documents**.

For more information on the types of KYC documents, please check <a href="https://developer.hipay.com/getting-started/platform-hipay-marketplace/KYC/" target="_blank">HiPay Marketplace - REST API to upload KYC documents</a>.
