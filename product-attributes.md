Below are the abstract steps for selecting product attributes from `Scayle` JSON response and showing them on our frontend.

- Firstly, we consume `Scayle` REST API on our `API Hub` using either product endpoints mentioned in their docs or their `Bapi SDK`
- Then we feed their JSON response to our mappers for the sake of transforming them into a simpler and more meaningful form.

```javascript
  static bapiProductAttributesToFrontasticAttributes = (
    attributesData, //response.attributes
  ) => {
    const attributes = {};

    for (const [key, attributeData] of Object.entries(attributesData)) {
      const attributeValues = [];

      if (attributeData.values instanceof Array) {
        for (const value of attributeData.values.values()) {
          attributeValues.push({
            key: value?.id,
            label: value?.label,
            value: value?.value,
            header: attributeData.label,
          });
        }
      } else {
        attributeValues.push({
          key: attributeData.values?.id,
          label: attributeData.values?.label,
          value: attributeData.values?.value,
          header: attributeData.label,
        });
      }

      attributes[key] = attributeValues;
    }

    return attributes;
  };

  static bapiProductAdvanceAttributesToFrontasticAttributes = (
        attributesData //response.advancedAttributes
      ) => {
    const attributes = {};

    for (const [key, attributeData] of Object.entries(attributesData)) {
      if (key == 'siblings') {
        continue;
      }

      const attributeValues = [];

      if (attributeData.values instanceof Array) {
        for (const value of attributeData.values.values()) {
          attributeValues.push({
            key: value?.fieldSet[0][0].id,
            label: value?.fieldSet[0][0].label,
            value: value?.fieldSet[0][0].value,
          });
        }
      }

      attributes[key] = attributeValues;
    }

    return attributes;
  };
```
- Now on our frontend we consumer our own REST API to fetch the product. And as you can see in the mappers above we shall receive product attributes in the following format
```json
// Notice that some or all of the following values might be empty
{
    "key": "string",
    "label": "string",
    "value": "string",
    "header": "string"
}
```
- Then we use our mappers on frontend to map our own API responses to responses that our components are expecting..
```typescript
export default class ProductMapper {
  static transformSizes(
    // will contain an attribute "attributes" including both
    // attributes and advanced attributes in the format
    // mentioned above
    variants,
  ) {
    const mappeVariants = [];
    variants?.forEach((variant) => {
      mappedVariants.push({
        description: this.getDescription(variant),
        passForm: this.getPassForm(variant),
        sustainability: this.getSustainability(variant),
        materials: this.getMaterials(variant)
      });
    });

    return mappedVariants;
  }

  //will select only a attribute from a given variant if it exists
  static getAttribute(obj, attribute) {
    //if the attribute exist add it to the description array
    const attr = obj.attributes?.[attribute];
    if (!!attr && Array.isArray(attr)) return attr.map((a) => a.label || a.value).join(', ');
    return attr?.label || attr?.value;
  }

  //select given attributes from a give variant if they exist
  static getAttributes(obj, attributes) {
    const selected = [];
    attributes.forEach((attribute) => {
      //if the attribute exist add it to the description array
      const attr =
        typeof attribute === 'string'
          ? this.getAttribute(obj, attribute)
          : (attribute as string[]).map((a) => this.getAttribute(obj, a)).filter((a) => !!a)[0];
      if (attr) selected.push(attr);
    });
    return selected;
  }

  static getAttributeHeader(obj, attribute) {
    //if the attribute exist add it to the description array
    const attr = obj.attributes?.[attribute];
    if (!!attr && Array.isArray(attr)) return attr[0].header;
    return attr?.header;
  }

  //Info getters
  static getDescription(variant) {
    if (!variant) return [];
    //the attributes to select
    const attributes = [
      'treatmentWashShortDescriptionGenericArticle',
      'sleeveLengthDescriptionGenericArticle',
      'necklineShortDescriptionGenericArticle',
      'collarShortDescriptionGenericArticle',
      'setShortDescriptionGenericArticle',
      'patternShortDescriptionColorLevel',
      'structureColorLevel',
      'pocketShortDescriptionColorLevel',
      'mainmaterialShortDescriptionColorLevel',
      'textStoffqualitaet',
      'treatmentWashShortDescriptionGenericArticle',
      'materialCompositionLiningShortDescriptionColorLevel',
      ['fastenerShortDescriptionColorLevel', 'textStoffqualitaetMaster', 'textStoffqualitaetProduct'],
    ];
    return this.getAttributes(variant, attributes);
  }

  static getPassForm(variant) {
    if (!variant) return [];
    //the attributes to select
    const attributes = [
      'legLengthShortDescriptionGenericArticle',
      'waistbandHeightShortDescriptionGenericArticle',
      'productDimensionsInCmVariant',
      'heelHeightShortDescriptionGenericArticle',
      'shaftweite',
      'shaftHeightShortDescriptionGenericArticle',
      'gurtelBreite',
      'kurzbeschreibung',
      'fallsSmallerBiggerColorLevel',
      'beltWidthShortDescriptionGenericArticle',
      'shaftWidthShortDescriptionGenericArticle',
    ];
    const fitDesc = this.getAttribute(variant, 'fitDescriptionGenericArticle');
    const fitShortDesc = this.getAttribute(variant, 'fitShortDescriptionGenericArticle');
    if (fitDesc && fitShortDesc) {
      return [`${fitDesc}: ${fitShortDesc}`, ...this.getAttributes(variant, attributes)];
    }
    return this.getAttributes(variant, attributes);
  }

  static getSustainability(variant) {
    if (!variant) return {};
    //the attributes to select
    const attribute = 'styleSustainableLongDescriptionColorLevel';
    return {
      subline:
        `BE PART - ` +
        (this.getAttribute(variant, 'hangtagSustainabilityCategory') ?? '') +
        ' : ' +
        (this.getAttribute(variant, 'hangtagSustainabilityDescription') ?? ''),
      description: this.getAttribute(variant, attribute),
    };
  }

  static getMaterials(variant) {
    if (!variant) return [];

    //the attributes to select
    const attributes = [
      'materialzusammensetung',
      'futter',
      'kurzbeschreibung',
      'materialCompositionLiningShortDescriptionColorLevel',
      'materialCompositionShellfabricColorLevel',
      'materialCompositionOuterLayerColorLevel',
      'materialCompositionInnerLayerColorLevel',
      'materialCompositionFrontPartColorLevel',
      'materialCompositionBackPartColorLevel',
      'materialCompositionSleeveColorLevel',
      'materialCompositionCollarColorLevel',
      'materialCompositionHoodColorLevel',
      'materialCompositionInsertColorLevel',
      'materialCompositionBasefabricColorLevel',
      'materialCompositionFurColorLevel',
      'materialCompositionLiningColorLevel',
      'materialCompositionLiningOutsideColorLevel',
      'materialCompositionLiningInsideColorLevel',
      'materialCompositionUpperLiningColorLevel',
      'materialCompositionLowerLiningColorLevel',
      'materialCompositionSleeveLiningColorLevel',
      'materialCompositionHoodLiningColorLevel',
      'materialCompositionInsertLiningColorLevel',
      'materialCompositionFillerColorLevel',
      'materialCompositionAccessorieColorLevel',
      'materialCompositionUpperPartColorLevel',
      'materialCompositionLowerPartColorLevel',
      'materialCompositionCoatingColorLevel',
      'materialCompositionBaseYarnColorLevel',
      'materialCompositionEffectYarnColorLevel',
      'materialCompositionInnerSoleColorLevel',
      'materialCompositionSoleColorLevel',
      'materialCompositionIngredientColorLevel',
      'materialCompositionFrontpartColorLevel',
      'materialCompositionBackpartColorLevel',
      'materialCompositionSleeveliningColorLevel',
      'materialCompositionHoodliningColorLevel',
    ];
    return attributes
      .map((a) => {
        const header = this.getAttributeHeader(variant, a);
        const value = this.getAttribute(variant, a);
        if (!(header && value)) return '';
        return `${header}: ${value}`;
      })
      .filter((a) => !!a);
  }
}
```
- Finally on our frontend we can do..
```javascript
    variant.description // Array of strings
    variant.passForm // Array of strings
    variant.sustainibility // Single string
    variant.materials // Array of strings
```