# optics

`optics` is a state management library for typescript apps based on [monocle-ts](https://github.com/gcanti/monocle-ts). It currently implements a single reducer and action creator, ready to be hooked in a standard redux setup. 

## ![image](https://media.tenor.com/images/74eae4ff92a933aaecf5b968aed5818d/tenor.gif)

Optics, more specifically lenses give you access to all getters and setters for every property of your state tree. Lenses are also composable so you can modify a deeply nested object property by passing a reference to the full object, and a function to update the specific property. All lenses operations are immutable.

## you're tired of writing reducers and selectors

As an example, let's say you want to retrieve some nested property from your state and show it in a view. Then you want to update it somehow. This is how you could write it with standard abstractions and using [redux-saga](https://github.com/redux-saga/redux-saga) :

```typescript

    /* ui/team-page/selectors.ts */

    export const selectUiTeamPageTeamName =
        (state: State) =>
            state.ui.teamPage.name
            
    /* ui/team-page/update-team-name.ts */

    export const updateUiTeamPageTeamName =
        (name: string) =>
            ({
                type: 'UI/TEAM_PAGE/UPDATE_TEAM_NAME',
                payload: name
            })

    export const uiTeamPageTeamNameUpdated =
        (name: string) =>
            ({
                type: 'UI/TEAM_PAGE/TEAM_NAME_UPDATED',
                payload: name
            })

    export const uiTeamPageTeamNameUpdateFailed =
        (name: string) =>
            ({
                type: 'UI/TEAM_PAGE/TEAM_NAME_UPDATE_FAILED',
                payload: name
            })

    export function* updateUiTeamPageTeamNameSaga({payload:{name}}: Action) {
        const saved = yield call(api, 'url', {name})
        if (saved.isSuccess)
            yield put(uiTeamPageTeamNameUpdated(name))
        else
            yield put(uiTeamPageTeamNameUpdateFailed(name))
    }

    export const uiTeamPageTeamNameReducer =
        (state: UiTeamPageStateTeamNameState, action: UpdatedAction) =>
            action.type === 'UI/TEAM_PAGE/UPDATE_TEAM_NAME'
            ? action.payload
            : state

    /* ui/team-page/update-team-name.test.ts */
    
    test('my saga actually works...', () => {...})

    test('I have to test reducers? what?', () => {
        expect(uiTeamPageTeamNameReducer(...)).toEqual(...)
    })

    /* ui/team-page/selectors.test.ts */

    test('I have to test selectors too?', () => {
        expect(selectUiTeamPageTeamName(...)).toEqual(...)
    })
    
    /* ui/team-page/index.ts */

    export const uiTeamPageReducer = combineReducers({
        teamName: uiTeamPageTeamNameReducer,
        ...otherReducers
    })
```

With optics, that would look more like:

```typescript

    /* lenses.ts */

    export const getUi =
        Lens.fromProp<State>()('ui')

    export const getTeamPage =
        Lens.fromProp<Ui>()('teamPage') 

    export const getUiTeamPageTeamName =
        Lens.fromProp<TeamPage>()('teamName')

    export const selectUiTeamPageTeamName =
        getUi
        .compose(getTeamPage)
        .compose(getTeamPageTeamName)

    /* ui/team-page/update-team-name.ts */

    export const updateUiTeamPageTeamName =
        (name: string) =>
            ({
                type: 'UI/TEAM_PAGE/UPDATE_TEAM_NAME',
                payload: name
            })

    export const uiTeamPageTeamNameUpdated =
        (name: string) =>
            ({
                type: 'UI/TEAM_PAGE/TEAM_NAME_UPDATED',
                payload: name
            })

    export const uiTeamPageTeamNameUpdateFailed =
        (name: string) =>
            ({
                type: 'UI/TEAM_PAGE/TEAM_NAME_UPDATE_FAILED',
                payload: name
            })

    export function* updateUiTeamPageTeamNameSaga({payload:{name}}: Action) {
        const saved = yield call(api, 'url', {name})
        if (saved.isSuccess)
            yield put(updateState(
                selectUiTeamPageTeamName.set(name)
            ))
            yield put(uiTeamPageTeamNameUpdated(name))
        else
            yield put(uiTeamPageTeamNameUpdateFailed(name))
    }

    /* ui/team-page/update-team-name.test.ts */
    
    test('my saga actually works...', () => {...})

    /***** No reducers here! *****/
```

Note that we don't have to write tests for the reducers or selectors on the second case, because there are no reducers, and no selectors, just lenses.
Another benefit we gained for free here is that we remove all the possible "logic" from our reducers. Some random dev wouldn't even be able to 
```typescript
    (state, action) =>
        if (state.shouldDoLogicInReducer) ? "no" : "nope"
```
and make your reducer really hard to understand.
Optics also makes you think more generically about your update functions, since they are the only thing you should test. If you plan on updating a collection with a new value, write a generic collection append function, test it once and never again think about writing tests again.

## but wait, I still have to write all those lenses...
<img src="https://media.giphy.com/media/l1KVaj5UcbHwrBMqI/source.gif" width="200" height="120" />

[not really](https://github.com/optics/optics-gen)

## Dependencies

You'll want to get [monocle-ts](https://github.com/gcanti/monocle-ts) and [fp-ts](https://github.com/gcanti/fp-ts).

`optics` also extend `monocle-ts` and introduces the concept of [projections](https://github.com/optics/projections), which is a peer of this library.

`optics` shouldn't actually depend on redux at all, but we do export everything as a reducer and action creator to make use of the great devtools redux has.

## Documentation

> Coming soon...

If you're looking for more information about optics in typescript, the [monocle-ts](https://github.com/gcanti/monocle-ts) website is a good start. More information will be added to the `optics` library as needed.

## install (to be published...)

`npm install optics`