https://gemini.google.com/app/4ab4dcce6bc66b04

The "OCI Referrers API" is a key part of the **Open Container Initiative (OCI) Distribution Specification v1.1**.1 Its primary purpose is to create a standardized way to **associate and discover artifacts** that are related to a container image in a registry.2

Before this API, if you wanted to attach metadata like a digital signature, a Software Bill of Materials (SBOM), or other attestations to a container image, you often had to resort to messy workarounds like using special naming conventions for tags.3 This made it difficult to manage, discover, and promote these related artifacts.

The OCI Referrers API solves this problem by creating a structured graph of relationships.4

### How It Works

The core of the OCI Referrers API relies on two main concepts:

- **`subject` field:** A new field was added to the OCI manifest specification.5 When you push an artifact (like an SBOM or signature) to a registry, you can include a `subject` field that points to the digest of the main container image it refers to.6 This creates the direct link.7
    
- **`/referrers` API endpoint:** This new API endpoint allows you to query a registry for all artifacts that reference a specific image digest.8 Instead of guessing a tag name, you can simply ask the registry, "Hey, what artifacts are related to this image?"9
    

### Key Benefits

The OCI Referrers API provides several significant benefits for modern software supply chains:10

- **Enhanced Security and Integrity:** It makes it simple and standardized to associate security-related artifacts like **signatures** and **SBOMs** with their corresponding images.11 This is crucial for verifying the integrity and origin of your software.
    
- **Improved Discoverability:** Related artifacts are no longer hidden behind obscure tag names.12 They are easily discoverable through a single, well-defined API call, which is essential for automated security scanning and policy enforcement.13
    
- **Simplified Management:** The API helps prevent "orphaned" artifacts.14 When you delete a container image, a compliant registry can automatically clean up the associated artifacts.15 Similarly, when you copy or promote an image, its related artifacts can be moved with it in a single operation.16
    
- **Increased Interoperability:** Because it's a part of the official OCI specification, different tools and platforms can all use the same, standardized method to manage and discover artifacts, which improves the entire container ecosystem.17
    

In essence, the OCI Referrers API transforms a container registry from just a simple storage location for images into a robust, interconnected system for managing a complete graph of artifacts and their relationships.