#This is how event loop works for client side in javascript

##An event loop must continually run through the following steps for as long as it exists:

> 1. Select the oldest task on one of the event loop's task queues, if any, ignoring, in the case of a browsing context event loop, tasks whose associated Documents are not fully active. The user agent may pick any task queue. If there is no task to select, then jump to the microtasks step below.

> 2. Set the event loop's currently running task to the task selected in the previous step.

> 3. Run: Run the selected task.

> 4. Set the event loop's currently running task back to null.

> 5. Remove the task that was run in the run step above from its task queue.

> 6. Microtasks: Perform a microtask checkpoint.

> 7. Update the rendering: If this event loop is a browsing context event loop (as opposed to a worker event loop), then run the following substeps.

Let now be the value that would be returned by the Performance object's now() method. [HRT]

Let docs be the list of Document objects associated with the event loop in question, sorted arbitrarily except that the following conditions must be met:

Any Document B that is nested through a Document A must be listed after A in the list.

If there are two documents A and B whose browsing contexts are both nested browsing contexts and their browsing context containers are both elements in the same Document C, then the order of A and B in the list must match the relative tree order of their respective browsing context containers in C.

In the steps below that iterate over docs, each Document must be processed in the order it is found in the list.

If there are top-level browsing contexts B that the user agent believes would not benefit from having their rendering updated at this time, then remove from docs all Document objects whose browsing context's top-level browsing context is in B.

Whether a top-level browsing context would benefit from having its rendering updated depends on various factors, such as the update frequency. For example, if the browser is attempting to achieve a 60Hz refresh rate, then these steps are only necessary every 60th of a second (about 16.7ms). If the browser finds that a top-level browsing context is not able to sustain this rate, it might drop to a more sustainable 30Hz for that set of Documents, rather than occasionally dropping frames. (This specification does not mandate any particular model for when to update the rendering.) Similarly, if a top-level browsing context is in the background, the user agent might decide to drop that page to a much slower 4Hz, or even less.

Another example of why a browser might skip updating the rendering is to ensure certain tasks are executed immediately after each other, with only microtask checkpoints interleaved (and without, e.g., animation frame callbacks interleaved). For example, a user agent might wish to coalesce timer callbacks together, with no intermediate rendering updates.
If there are a nested browsing contexts B that the user agent believes would not benefit from having their rendering updated at this time, then remove from docs all Document objects whose browsing context is in B.

As with top-level browsing contexts, a variety of factors can influence whether it is profitable for a browser to update the rendering of nested browsing contexts. For example, a user agent might wish to spend less resources rendering third-party content, especially if it is not currently visible to the user or if resources are constrained. In such cases, the browser could decide to update the rendering for such content infrequently or never.

For each fully active Document in docs, run the resize steps for that Document, passing in now as the timestamp. [CSSOMVIEW]

For each fully active Document in docs, run the scroll steps for that Document, passing in now as the timestamp. [CSSOMVIEW]

For each fully active Document in docs, evaluate media queries and report changes for that Document, passing in now as the timestamp. [CSSOMVIEW]

For each fully active Document in docs, run CSS animations and send events for that Document, passing in now as the timestamp. [CSSANIMATIONS]

For each fully active Document in docs, run the fullscreen rendering steps for that Document, passing in now as the timestamp. [FULLSCREEN]

⋰
Spec bugs: 28001
For each fully active Document in docs, run the animation frame callbacks for that Document, passing in now as the timestamp.

For each fully active Document in docs, run the update intersection observations steps for that Document, passing in now as the timestamp. [INTERSECTIONOBSERVER]

For each fully active Document in docs, update the rendering or user interface of that Document and its browsing context to reflect the current state.

If this is a worker event loop (i.e. one running for a WorkerGlobalScope), but there are no tasks in the event loop's task queues and the WorkerGlobalScope object's closing flag is true, then destroy the event loop, aborting these steps, resuming the run a worker steps described in the Web workers section below.

Return to the first step of the event loop.
