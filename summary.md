Our datacompass application is a CAT 3 application classified as non-highly confidential data, non-HCD. 
We currently crawl highly confidential category hcd workspaces based on an exception we have been able to obtain.
We have a requirement to crawl the metadata for CCB risk PCI workspaces in Databricks . PCI is a higher level of category above the HCD. According to the restrictions 
- cat 1 seal will be treated as PCI and cat3 seals will be broken down to hcd and non HD and treated as non-pci. 
- CAT1 PCI seals cannot be onboarded to non-PCI workspaces, whereas CAT3 non-PCI seals cannot be onboarded to PCI workspaces.
- Databricks does not support CAT2 seal onboarding. Please reach out to PCI teams to discuss your use case.
-