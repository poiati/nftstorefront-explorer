<script>
  import * as fcl from "@onflow/fcl";

  fcl.config({
    "accessNode.api": "https://access-mainnet-beta.onflow.org",
    "0xNFTStoreFront": "0x4eb8a10cb9f87357",
  });

  let networks = ["mainnet", "testnet"];
  let selectedNetwork = networks[0];

  let accountAddress = "";

  let result = [];

  $: isAccountAddressValid = accountAddress !== "";

  const sendQuery = async () => {
    const value = await fcl.query({
      cadence: `
        import NFTStorefront from 0xNFTStoreFront

        pub struct ResponseData {
          pub let found: Bool
          pub let listings: [NFTStorefront.ListingDetails]?

          init(found: Bool, listings: [NFTStorefront.ListingDetails]?) {
            self.found = found
            self.listings = listings
          }
        }

        pub fun main(storefrontAccountAddress: Address): ResponseData {
          let storefrontAccount = getAccount(storefrontAccountAddress)
          let storefrontRef = storefrontAccount.getCapability<&NFTStorefront.Storefront{NFTStorefront.StorefrontPublic}>(
            NFTStorefront.StorefrontPublicPath
          ).borrow()

          if storefrontRef == nil {
            return ResponseData(found: false, listings: nil)
          }
              
          let listingIDs = storefrontRef!.getListingIDs()
          let listingsDetails: [NFTStorefront.ListingDetails] = []

          for listingID in listingIDs {
            let listing = storefrontRef!.borrowListing(listingResourceID: listingID)!
            listingsDetails.append(listing.getDetails())
          }

          return ResponseData(found: true, listings: listingsDetails)
        }
      `,
      args: (arg, t) => [arg("0x12450e4bb3b7666e", t.Address)],
    });

    console.log(value);
  };
</script>

<main>
  <h1 class="text-3xl font-bold">NFTStoreFront Explorer</h1>

  <form>
    <div>
      <label for="network">Network:</label>

      <select value={selectedNetwork}>
        {#each networks as network}
          <option value={network}>{network}</option>
        {/each}
      </select>
    </div>

    <div>
      <label for="accountAddress">Account Address:</label>
      <input type="text" bind:value={accountAddress} />
    </div>

    <div>
      <button
        disabled={!isAccountAddressValid}
        on:click|preventDefault={sendQuery}>Explore</button
      >
    </div>
  </form>

  {#if result.length > 0}
    <section class="result">
      <table>
        <thead>
          <th>Listing ID</th>
        </thead>
        <tbody>
          <tr>10</tr>
        </tbody>
      </table>
    </section>
  {/if}
</main>

<style>
  form {
    display: flex;
    gap: 1em;
  }

  .result {
    margin-top: 1em;
  }

  table {
    border: 0;
    border-spacing: 0;
  }
</style>
