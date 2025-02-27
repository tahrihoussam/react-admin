---
layout: default
title: "Realtime"
---

# Realtime

React-admin provides hooks and UI components for collaborative applications where several people work in parallel. It allows publishing and subscribing to real-time events, updating views when another user pushes a change, notifying end users of events, and preventing data loss when two editors work on the same resource concurrently.

<video controls autoplay muted>
  <source src="./img/CollaborativeDemo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

These features are provided by the `ra-realtime` package, which is part of the [Enterprise Edition](https://marmelab.com/ra-enterprise)<img class="icon" src="./img/premium.svg" /> 

## Backend Agnostic

The `ra-realtime` package supports various realtime infrastructures:  

- [Mercure](https://mercure.rocks/),
- [API Platform](https://api-platform.com/docs/admin/real-time-mercure/#real-time-updates-with-mercure),
- [supabase](https://supabase.com/),
- [Socket.IO](https://socket.io/),
- [Ably](https://ably.com/),
- and many more.

That's because it uses the same _adapter_ approach as for CRUD methods. In fact, the `dataProvider` is used to send and receive events.

See the [Data Provider Requirements](./RealtimeDataProvider.md) page for more information.

## Publish/Subscribe

At its core, `ra-realtime` provides a **pub/sub mechanism** to send and receive real-time events. Events are sent to a topic, and all subscribers to this topic receive the event.

```jsx
// on the publisher side
const [publish] = usePublish();
publish(topic, event);

// on the subscriber side
useSubscribe(topic, callback);
```

`ra-realtime` provides a set of high-level hooks to make it easy to work with real-time events:

-   [`usePublish`](./usePublish.md)
-   [`useSubscribe`](./useSubscribe.md)
-   [`useSubscribeCallback`](./useSubscribeCallback.md)
-   [`useSubscribeToRecord`](./useSubscribeToRecord.md)
-   [`useSubscribeToRecordList`](./useSubscribeToRecordList.md)

## Live Updates

Ra-realtime provides **live updates** via specialized hooks and components. This means that when a user edits a resource, the other users working on the same resource see the changes in real-time whether they are in a list, a show view, or an edit view.

For instance, replace `<List>` with `<ListLive>` to have a list refreshing automatically when an element is added, updated, or deleted:

```diff
import {
-   List,
    ListProps,
    Datagrid,
    TextField,
    NumberField,
    Datefield,
} from 'react-admin';
+import { ListLive } from '@react-admin/ra-realtime';

const PostList = (props: ListProps) => (
-   <List {...props}>
+   <ListLive {...props}>
        <Datagrid>
            <TextField source="title" />
            <NumberField source="views" />
            <DateField source="published_at" />
        </Datagrid>
-   </List>
+   </ListLive>
);
```

![useSubscribeToRecordList](./img/useSubscribeToRecordList.gif)

This feature leverages the following hooks:

-   [`useGetListLive`](./useGetListLive.md)
-   [`useGetOneLive`](./useGetOneLive.md)

And the following components:

-   [`<ListLive>`](./ListLive.md)
-   [`<EditLive>`](./EditLive.md)
-   [`<ShowLive>`](./ShowLive.md)

## Menu Badges

Ra-realtime also provides **badge notifications in the Menu**, so that users can see that something new happened to a resource list while working on another one.

![MenuLive](./img/RealtimeMenu.png)

Use `<MenuLive>` instead of react-admin's `<Menu>` to get this feature:

```jsx
import React from 'react';
import { Admin, Layout, Resource } from 'react-admin';
import { MenuLive } from '@react-admin/ra-realtime';

import { PostList, PostShow, PostEdit, realTimeDataProvider } from '.';

const CustomLayout = (props) => (
    <Layout {...props} menu={MenuLive} />
);

const MyReactAdmin = () => (
    <Admin dataProvider={realTimeDataProvider} layout={CustomLayout}>
        <Resource name="posts" list={PostList} show={PostShow} edit={PostEdit} />
    </Admin>
);
```

This feature leverages the following components:

-   [`<MenuLive>`](./MenuLive.md)
-   [`<MenuLiveItemLink>`](./MenuLive.md)

## Locks

And last but not least, ra-realtime provides a **lock mechanism** to prevent two users from editing the same resource at the same time.

![Edit With Locks](./img/locks-demo.gif)

A user can lock a resource, either by voluntarily asking for a lock or by editing a resource. When a resource is locked, other users can't edit it. When the lock is released, other users can edit the resource again.

```jsx
export const NewMessageForm = () => {
    const [create, { isLoading: isCreating }] = useCreate();
    const record = useRecordContext();

    const { data: lock } = useGetLockLive('tickets', { id: record.id });
    const { identity } = useGetIdentity();
    const isFormDisabled = lock && lock.identity !== identity?.id;

    const [doLock] = useLockOnCall({ resource: 'tickets' });
    const handleSubmit = (values: any) => {
        /* ... */
    };

    return (
        <Form onSubmit={handleSubmit}>
            <TextInput
                source="message"
                multiline
                onFocus={() => {
                    doLock();
                }}
                disabled={isFormDisabled}
            />
            <SelectInput
                source="status"
                choices={statusChoices}
                disabled={isFormDisabled}
            />
            <Button type="submit" disabled={isCreating || isFormDisabled}>
                Submit
            </Button>
        </Form>
    );
};
```

This feature leverages the following hooks:

-   [`useLock`](./useLock.md)
-   [`useUnlock`](./useUnlock.md)
-   [`useGetLock`](./useGetLock.md)
-   [`useGetLockLive`](./useGetLockLive.md)
-   [`useGetLocks`](./useGetLocks.md)
-   [`useGetLocksLive`](./useGetLocksLive.md)
-   [`useLockOnCall`](./useLockOnCall.md)
-   [`useLockOnMount`](./useLockOnMount.md)

## Installation

```sh
npm install --save @react-admin/ra-realtime
# or
yarn add @react-admin/ra-realtime
```

`ra-realtime` is part of the [React-Admin Enterprise Edition](https://marmelab.com/ra-enterprise/), and hosted in a private npm registry. You need to subscribe to one of the Enterprise Edition plans to install this package.

You will need a data provider that supports real-time subscriptions. Check out the [Data Provider Requirements](./RealTimeDataProvider.md) section for more information.

## I18N

This module uses specific translations for displaying notifications. As for all translations in react-admin, it's possible to customize the messages.

To create your own translations, you can use the TypeScript types to see the structure and see which keys are overridable.

Here is an example of how to customize translations in your app:

```jsx
import polyglotI18nProvider from 'ra-i18n-polyglot';
import englishMessages from 'ra-language-english';
import frenchMessages from 'ra-language-french';
import {
    TranslationMessages as BaseTranslationMessages,
    raRealTimeEnglishMessages,
    raRealTimeFrenchMessages,
    RaRealTimeTranslationMessages,
} from '@react-admin/ra-realtime';

/* TranslationMessages extends the defaut translation
 * Type from react-admin (BaseTranslationMessages)
 * and the ra-realtime translation Type (RaRealTimeTranslationMessages)
 */
interface TranslationMessages
    extends RaRealTimeTranslationMessages,
        BaseTranslationMessages {}

const customEnglishMessages: TranslationMessages = mergeTranslations(
    englishMessages,
    raRealTimeEnglishMessages,
    {
        'ra-realtime': {
            notification: {
                record: {
                    updated: 'Wow, this entry has been modified by a ghost',
                    deleted: 'Hey, a ghost has stolen this entry',
                },
            },
        },
    }
);

const i18nCustomProvider = polyglotI18nProvider(locale => {
    if (locale === 'fr') {
        return mergeTranslations(frenchMessages, raRealTimeFrenchMessages);
    }
    return customEnglishMessages;
}, 'en');

export const MyApp = () => (
    <Admin dataProvider={myDataprovider} i18nProvider={i18nCustomProvider}>
        ...
    </Admin>
);
```