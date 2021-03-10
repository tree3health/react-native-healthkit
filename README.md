# @kingstinct/react-native-healthkit

[![Test](https://github.com/Kingstinct/react-native-healthkit/actions/workflows/main.yml/badge.svg)](https://github.com/Kingstinct/react-native-healthkit/actions/workflows/main.yml)
[![hello](https://img.shields.io/npm/v/@kingstinct/react-native-healthkit?color=red)](https://www.npmjs.com/package/@kingstinct/react-native-healthkit)

React Native bindings for HealthKit with full TypeScript and Promise support covering almost everything. Keeping TypeScript mappings as close as possible to HealthKit - both in regards to naming and serialization. This will make it easier to keep this library up-to-date with HealthKit as well as browsing the official documentation (and if something - metadata properties for example - is not typed it will still be accessible).

| Data Types                  | Query | Save  | Subscribe | Examples                               |
| ----------------------------|:------|:------|:----------|:---------------------------------------|
| 80+ Quantity Types          | ✅    | ✅    | ✅       | Steps, energy burnt, blood glucose etc.. |
| 13 Category Types           | ✅    | ✅    | ✅       | Sleep analysis, mindful sessions etc.. |
| 75+ Workout Activity Types  | ✅    | ✅    | ✅       | Swimming, running, table tennis etc..  |
| Correlation Types           | ✅    | ✅    | ✅       | Food and blood pressure                |
| Document Types              | ✅    | ❌    | ✅       | [CDA documents](https://developer.apple.com/documentation/healthkit/hkcdadocument) exposed as Base64 data  |
| Clinical Records            | ✅    | ❌    | ✅       | Lab results etc in [FHIR JSON format](https://www.hl7.org/fhir/json.html)  |

## Installation

```sh
npm install @kingstinct/react-native-healthkit
cd ios
pod install
```

or

```sh
yarn add @kingstinct/react-native-healthkit
cd ios
pod install
```

Remember to set `NSHealthUpdateUsageDescription` and `NSHealthShareUsageDescription` in your `Info.plist` as well as enable the HealthKit capability for the project in Xcode. During runtime you need to check if HealthKit is available on the device with `isHealthDataAvailable` and request permissions with `requestAuthorization`;

Since this package is using Swift you might also need to add a bridging header in your project if you haven't already, you can [find more about that in the official React Native docs](https://reactnative.dev/docs/native-modules-ios#exporting-swift)

## Usage

```TypeScript
  import HealthKit, { HKUnit, HKQuantityTypeIdentifier, HKInsulinDeliveryReason, HKCategoryTypeIdentifier } from '@kingstinct/react-native-healthkit';

  const isAvailable = await HealthKit.isHealthDataAvailable();

  /* Read latest sample of any data */
  await HealthKit.requestAuthorization([HKQuantityTypeIdentifier.bodyFatPercentage]); // request read permission for bodyFatPercentage

  const { quantity, unit, startDate, endDate } = await HealthKit.getMostRecentQuantitySample(HKQuantityTypeIdentifier.bodyFatPercentage); // read latest sample
  
  console.log(quantity) // 17.5
  console.log(unit) // %

  /* Listen to data */
  await HealthKit.requestAuthorization([HKQuantityTypeIdentifier.heartRate]); // request read permission for heart rate

  const unsubscribe = HealthKit.subscribeToChanges(HKQuantityTypeIdentifier.heartRate, () => {
    // refetch whichever queries you need
  });

  /* write data */
  await HealthKit.requestAuthorization([], [HKQuantityTypeIdentifier.insulinDelivery]); // request write permission for insulin delivery

  ReactNativeHealthkit.saveQuantitySample(
      HKQuantityTypeIdentifier.insulinDelivery,
      HKUnit.InternationalUnit,
      5.5,
      {
        metadata: {
          // Metadata keys could be arbirtary string to store app-specific data.
          // To use built-in types from https://developer.apple.com/documentation/healthkit/samples/metadata_keys
          // you need to specify string values instead of variable names (by dropping MetadataKey from the name).
          HKInsulinDeliveryReason: HKInsulinDeliveryReason.basal,
        },
      }
    );


  /* Hooks */
  const lastBodyFatSample = HealthKit.useMostRecentQuantitySample(HKQuantityTypeIdentifier.bodyFatPercentage);
  const lastMindfulSession = Healthkit.useMostRecentCategorySample(
    HKCategoryTypeIdentifier.mindfulSession
  );
  const lastWorkout = Healthkit.useMostRecentWorkout();
```

## A note on Apple Documentation

We're striving to do as straight a mapping as possible to the Native Libraries. This means that in most cases the Apple Documentation makes sense. However, when it comes to the Healthkit [Metadata Keys](https://developer.apple.com/documentation/healthkit/samples/metadata_keys) the documentation doesn't actually reflect the serialized values. For example HKMetadataKeyExternalUUID in the documentation serializes to HKExternalUUID - which is what we use.

## Clinical Records

For accessing Clinical Records use old version (3.x) or use specific branch ""

## Contributing

See the [contributing guide](CONTRIBUTING.md) to learn how to contribute to the repository and the development workflow.

## License

MIT
