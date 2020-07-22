---
layout: post
title: Store Microsoft Teams SPFx Personal App Properties
tags:
  - MS Graph
  - MS Teams
  - Microsoft 365
  - Microsoft 365 Development
  - Microsoft Graph
  - Microsoft Teams
  - SPFx
  - SPFx Web Parts
  - SharePoint
  - SharePoint Development
  - SharePoint Framework
  - Teams
  - Personal App
featured_image: assets/images/posts/2020/teams-personal-app-settings.png
fearured_image_thumbnail: assets/images/posts/2020/teams-personal-app-settings.png
---
Teams Personal Apps, or Personal Tabs don't have settings. It means that by default Personal Apps show similar content for all users. The only dynamic to change the content of a tab is the current user.
For SharePoint Framework Personal App (Web Part) it means a few things:
- Web Part will never be switched to Edit mode
- Property Pane will never be shown
- `this.properties` value is always `undefined`

But there are scenarios when we want to configure Personal App and store this configuration on per user basis.
There are multiple different approaches how we can store web part's properties in such situation:
1. Store properties as JSON files in user's OneDrive. This approach is described by Robert Schouten [here](https://robertschouten.com/2020/05/19/microsoft-teams-spfx-personal-app-configuration/).
2. Microsoft Graph Open Extensions and Schema Extensions [https://docs.microsoft.com/en-us/graph/extensibility-overview](https://docs.microsoft.com/en-us/graph/extensibility-overview).
3. Some global list. For example, on root SharePoint site.
4. External storage like Azure.
5. ...

In this post I want to describe an approach that is similar to #1 but uses list instead of document library. By the way, Microsoft uses similar approach to store manifests for SPFx components (web parts, extensions, library components) in SharePoint App Catalog.
You can find the code for this approach [here](https://github.com/pnp/sp-dev-fx-webparts/tree/master/samples/react-teams-personal-app-settings).

## Custom Property Pane
Regardless of what approach you select to store the proerties you will still need to implement custom property pane.
Fortunately, it's an easy task as we can use [Panel](https://developer.microsoft.com/en-us/fluentui#/controls/web/panel) component from Fluent UI Framework.
You can render inputs, checkboxes, dropdowns, etc. in these panel, Save and Cancel buttons to make it look similar to OOTB SPFx Property Pane.
Again, I would recommend using available [Fluent UI](https://developer.microsoft.com/en-us/fluentui#/controls/web) controls or [PnP Reusable Controls](https://pnp.github.io/sp-dev-fx-controls-react/).
Here is a simple implementation for custom property pane that renders text inputs for title and description, and also update the properties stored in [React Context](https://reactjs.org/docs/context.html).
```typescript
import * as React from 'react';
import { Panel } from 'office-ui-fabric-react/lib/Panel';
import { TextField } from 'office-ui-fabric-react/lib/TextField';
import { PrimaryButton, DefaultButton } from 'office-ui-fabric-react/lib/Button';
import AppContext from '../../common/AppContext';
import * as strings from 'PersonalAppSettingsWebPartStrings';

/**
 * Component props
 */
export interface ISettingsPanelProps {
  /**
   * Panel close handler
   */
  onClosePanel: () => void;
}

/**
 * Component to update web part props
 */
export const SettingsPanel: React.FunctionComponent<ISettingsPanelProps> = (props: ISettingsPanelProps) => {
  // getting context
  const { webPartProps, onUpdateProps } = React.useContext(AppContext);
  // title value
  const [title, setTitle] = React.useState<string>(webPartProps ? webPartProps.title : '');
  // description value
  const [description, setDescription] = React.useState<string>(webPartProps ? webPartProps.description : '');

  /**
   * save button click handler
   */
  const save = () => {
    onUpdateProps({
      title: title,
      description: description
    });

    props.onClosePanel();
  };

  /**
   * Cancel button click handler
   */
  const cancel = () => {
    props.onClosePanel();
  };

  /**
   * Renders panel footer content
   */
  const onRenderFooter = () => {
    return <div>
      <PrimaryButton text={strings.Save} onClick={save} />
      <DefaultButton text={strings.Cancel} onClick={cancel} />
    </div>;
  };

  return (
    <Panel
      headerText={strings.WebPartSettings}
      isOpen={true}
      onRenderFooterContent={onRenderFooter}
    >
      <TextField
        label={strings.WebPartTitle}
        value={title || ''}
        onChange={(e, v) => { setTitle(v); }}>
      </TextField>
      <TextField
        label={strings.WebPartDescription}
        value={description || ''}
        onChange={(e, v) => { setDescription(v); }}>
      </TextField>
    </Panel>
  );
};
```

## OneDrive List Structure
Let's define the structure of the list to store the properties.
We need a column for the properties JSON and, if we want to store settings for multiple web parts, we need a column for unique key of the web part:
| column | type | role |
| ------ | ---- | ---- |
| WPKey | Single line of text | Stores web part's unique key |
| WPProperties | Multiple lines of text | Stores web part's properties |

## Service to Work with Properties
Now let's implement a service to read and write properties to the user's OneDrive custom list.
Our service should `get` properties from OneDrive and `set` properties to OneDrive. We can also make this service universal (for any type of web part properties) by using generic `T` type for the properties parameters.
Let's define the interface for the service:
```typescript
/**
 * Interface to work with web part properties
 */
export interface IWebPartPropertiesService<T> {
  /**
   * Gets web part properties
   */
  getProperties: (webPartKey: string) => Promise<T | null>;
  /**
   * Sets web part properties
   */
  setProperties: (webPartKey: string, properties: T) => Promise<void>;
}
```
This interface has no relation to OneDrive or any other implementation. So you can add multiple implementations for different approaches listed above to play with them or use them based on use case.
Now let's implement this interface in `OneDriveListWebPartPropertiesService` class.
In the service we will not only get or set properties but we'll also ensure the needed list (check if exists and create a new one if not):
```typescript
const PropertiesListTitle = 'WPProperties';
const MySiteGraphIdStorageKey = 'MySiteId';
const SettingsListIdStorageKey = 'SettingsListId';

/**
 * Gets MS Graph site ID for current user's OneDrive site
 */
private async _getMySiteGraphId(): Promise<string> {
  // we can cache the ID in the localStorage as it will never change for current user
  let graphSiteId = window.localStorage.getItem(MySiteGraphIdStorageKey);
  if (!graphSiteId) {

    const graphClient = await this._context.msGraphClientFactory.getClient();
    const currentDomain = location.hostname;
    const oneDriveDomain = `${currentDomain.split('.')[0]}-my.sharepoint.com`;

    const sharepointIdsResponse = await graphClient.api('/me/drive/root?$select=sharepointIds').version('v1.0').get();
    const sharepointIds = sharepointIdsResponse.sharepointIds;

    graphSiteId = `${oneDriveDomain},${sharepointIds.siteId},${sharepointIds.webId}`;

    window.localStorage.setItem(MySiteGraphIdStorageKey, graphSiteId);
  }

  return graphSiteId;
}

/**
 * Gets settings list id
 */
private async _getSettingListId(): Promise<string | null> {
  // we can cache the ID in the localStorage as it will never change
  let listId = window.localStorage.getItem(SettingsListIdStorageKey);
  if (!listId) {
    const graphSiteId = await this._getMySiteGraphId();

    const graphClient = await this._context.msGraphClientFactory.getClient();
    const listsResponse = await graphClient.api(`/sites/${graphSiteId}/lists?$filter=displayName eq '${PropertiesListTitle}'`).version('v1.0').get();

    if (listsResponse.value && listsResponse.value.length) {
      listId = listsResponse.value[0].id;
      window.localStorage.setItem(SettingsListIdStorageKey, listId!);
    }
    else {
      // creating the list if it hasn't been created before
      try {
        const createListResponse = await graphClient.api(`/sites/${graphSiteId}/lists`).version('v1.0').post({
          displayName: PropertiesListTitle,
          columns: [{
            name: WebPartUniqueKeyColumnName,
            text: {}
          }, {
            name: PropertiesColumnName,
            text: {
              allowMultipleLines: true,
              maxLength: 1000000000,
              textType: 'plain'
            }
          }],
          list: {
            hidden: true,
            template: 'genericList'
          }
        });

        listId = createListResponse.id;
        window.localStorage.setItem(SettingsListIdStorageKey, listId!);
      }
      catch (error) {
        if (error.statusCode === 403 || error.accessCode === 'accessDenied') {
          throw Error(strings.SiteManagePermissionsNotProvisioned);
        }
      }
    }

  }

  return listId;
}
```

`this._context` here is a Web Part's context (`WebPartContext`).

We need to implement one more helper method - we need to check if a list item for this web part has already been created. This check shows if there are any previously saved properties for this web part.
```typescript
/**
 * Gets list item with previously saved properties
 * @param webPartKey web part unique key
 * @param expandFields flag to expand fields
 */
private async _getPropertiesListItem(webPartKey: string, expandFields: boolean): Promise<IListItem | null | undefined> {
  const listId = await this._getSettingListId();
  if (!listId) {
    throw Error(strings.PropertiesListNotCreatedError);
  }

  const graphSiteId = await this._getMySiteGraphId();

  const graphClient = await this._context.msGraphClientFactory.getClient();

  let expandQuery = '';
  if (expandFields) {
    expandQuery = `&expand=fields`;
  }

  const existingItemResponse = await graphClient.api(`/sites/${graphSiteId}/lists/${listId}/items?select=id${expandQuery}`).version('v1.0').get();
  if (existingItemResponse.value && existingItemResponse.value.length && expandFields) {
    return existingItemResponse.value.filter(v => v.fields[WebPartUniqueKeyColumnName] === webPartKey)[0];
  }

  return null;
}
```

Now we can implement the interface itself by adding methods to get and set web part's properties:
```typescript
/**
 * Gets properties for the web part base on unique key (Properties OneDrive list can contain properties of multiple web parts).
 * @param webPartKey The key of the web part to get properties for.
 */
public async getProperties(webPartKey: string): Promise<T | null> {
  if (!this._properties) {
    // if there are no cached properties, we're getting them from the OneDrive
    const listItem = await this._getPropertiesListItem(webPartKey, true);

    // checking if there are any previously saved properties
    if (listItem) {
      this._properties = JSON.parse(listItem.fields![PropertiesColumnName]);
    }
  }
  return this._properties;
}

/**
 * Sets properties for the web part base on unique key (Properties OneDrive list can contain properties of multiple web parts).
 * @param webPartKey The key of the web part to get properties for.
 */
public async setProperties(webPartKey: string, properties: T): Promise<void> {
  // getting list id
  const listId = await this._getSettingListId();

  if (!listId) {
    // no list found
    throw Error(strings.PropertiesListNotCreatedError);
  }

  // updating internal cache
  this._properties = JSON.parse(JSON.stringify(properties));

  // converting properties object to a string
  const propertiesStr = JSON.stringify(properties);

  // getting graph site id (<tenant,site-id,web-id>) to work with
  const graphSiteId = await this._getMySiteGraphId();

  // getting graph client
  const graphClient = await this._context.msGraphClientFactory.getClient();

  // checking if there are previously saved properties
  const existingItem = await this._getPropertiesListItem(webPartKey, true);
  if (existingItem) {
    //
    // updaging properties
    //
    const itemId = existingItem.id;
    let fields: any = {};
    fields[PropertiesColumnName] = propertiesStr;
    const updateItemResponse = await graphClient.api(`/sites/${graphSiteId}/lists/${listId}/items/${itemId}/fields`).version('v1.0').patch(fields);

    if (updateItemResponse.error) {
      throw new Error(strings.PropertiesNotSavedError);
    }
  }
  else {
    //
    // saving properties for the first time
    //
    let fields: any = {};
    fields[WebPartUniqueKeyColumnName] = webPartKey;
    fields.Title = webPartKey;
    fields[PropertiesColumnName] = propertiesStr;
    const createItemResponse = await graphClient.api(`/sites/${graphSiteId}/lists/${listId}/items`).version('v1.0').post({
      fields: fields
    });

    if (createItemResponse.error) {
      throw new Error(strings.PropertiesNotSavedError);
    }
  }
}
```

## Use the Service in the Web Part
Usage of the service is pretty simple.
First, we need to add fields to store properties and the service instance. We can't use `this.properties` property of the web part - it will always be undefined, even if we set it to some custom value.
```typescript
private _props: IWebPartProps | null;
private _webPartPropertiesService: IWebPartPropertiesService<IWebPartProps>;
```

Next, we need to get current properties of the web part in `onInit`:
```typescript
public async onInit(): Promise<any> {
  this.context.statusRenderer.displayLoadingIndicator(this.domElement, strings.Loading);
  this._webPartPropertiesService = new OneDriveListWebPartPropertiesService<IWebPartProps>(this.context);
  try {
    this._props = await this._webPartPropertiesService.getProperties(WebPartKey);
  }
  catch (err) {
    this.context.statusRenderer.clearLoadingIndicator(this.domElement);
    this.renderError(err);
  }
}
```

And the last one - save the updated properties when needed:
```typescript
private _onUpdateProps = async (webPartProps: IWebPartProps): Promise<void> => {
  this._props = webPartProps;
  try {
    await this._webPartPropertiesService.setProperties(WebPartKey, webPartProps);
    this.render();
  }
  catch (err) {
    this.renderError(err);
  }
}
```

## Conclusion
As you can see, the implementation is pretty simple but yet universal. If you develop multiple Personal apps you can store all the properties for all of them in the same single OneDrive list.
As mentioned in the beginning of the post you can find the code for this post [here](https://github.com/pnp/sp-dev-fx-webparts/tree/master/samples/react-teams-personal-app-settings).
You can also find a Community demo for this sample [here](https://www.youtube.com/watch?v=vC4_cMrVx0o).

<br />
That's all for today!<br />
Have fun!