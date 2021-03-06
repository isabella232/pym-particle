Pym Particle
============

**NOTE: This repo is now under development as Sidechain at https://github.com/nprapps/sidechain! Please follow and work with us there.**

Pym Particle is a responsive iframe solution that's compatible with Pym v1 for basic height management, but with a simpler core built around newer JavaScript standards:

* Uses transferable JSON for messages, instead of a proprietary string
* Based on Custom Elements v1
* Uses shadow DOM to isolate the iframe from parent page styling
* Smaller API surface area
* AMP-compatible on both sides

If you're using this in a browser that supports custom elements, you should be able to create a host frame by simply adding the correct element to your page:

.. code-block:: html

    <pym-particle src="https://your-embed-here"></pym-particle>

On the other side, embedded pages should call ``PymParticle.registerGuest()`` to start sending height events to the parent. ``registerGuest()`` returns a GuestParticle instance, which supports Pym v1 ``on()`` event listeners for legacy events. However, we recommend that guest and host pages using Pym Particle use ``sendMessage()`` (a convenience wrapper around ``window.postMessage()``) to transmit, and register their own message handlers on the window for the receiving end.

Trying it out
-------------

This repo is just for demo purposes--at some point, we'll end up refactoring into a more traditional library structure. However, the current demo provides a useful environment for running a demo page and is based on our `interactive template <https://github.com/nprapps/interactive-template>`_.

To try out the code and make some tests of your own, clone this repo, ``npm install`` to get dependencies, and then run ``grunt`` to start the build process. If you do not have Grunt installed, run ``npm i grunt-cli -g`` to get the command-line shim first. Finally, visit ``localhost:8000`` in your browser to view the test page. 

Frequently Asked Questions
--------------------------

Is this a replacement for the existing Pym?
  **No, not at this time.** This project is a thought experiment on what Pym would look like if we wrote it today, keeping the core small and leveraging the features built-in to modern browsers. It is also a way to open a conversation with community members about what they want and need from an updated version of Pym.

Does this work in all browsers?
  No, this is intended only for modern browsers--those that support the Custom Elements v1 spec (and optionally, Shadow DOM). That means Chrome, Firefox, and Safari >= 10.1. Edge works just fine if you load the `Custom Elements polyfill <https://github.com/webcomponents/custom-elements>`_ first, although this adds about 20KB to the unminified bundle size, so you may want to feature-test or use their dynamic loader.

How do I scroll to an element in Pym Particle?
  Instead of offering a ``scrollParentToChildPos()``, requiring you to compute the offset of the element, use the browser's native ``Element.scrollIntoView()`` method (`documentation on MDN <https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView>`_).

How do I navigate the parent page?
  If possible, for accessibility reasons, page navigations should be exposed as links. Use the ``target="_parent"`` attribute to ask the parent page to navigate. If you need to navigate programmatically, you may need to write a custom message handler for it--Pym Particle does not make assumptions about how your single-page app handles routing.

Does Pym Particle provide arbitrary messaging support?
  Not really. Sending messages using ``window.postMessage()`` between guest and host pages is simple enough that it does not make sense to provide additional layers of abstraction. Guest/host instances do provide a ``sendMessage()`` method just for convenience (it's easier than having to search for and access each iframe's ``contentWindow``), but that's it. We will, however, make available a loader library that demonstrates some useful functionality, such as firing visibility events and passing data between frames.

Code snippets
-------------

.. code-block:: javascript

    // sending a message to an individual child
    // the sentinel serves as a way to test on the other side
    var particle = document.querySelector("pym-particle.individual");
    particle.sendMessage({
      sentinel: "npr",
      type: "log",
      message: "Hello from NPR"
    });

    // receiving a message in the child
    window.addEventListener("message", function(e) {
      if (e.data.sentinel && e.data.sentinel == "npr") {
        switch (e.data.type) {
          case "log":
            console.log(e.data.message); // Hello from NPR
            break;

          default:
            console.warn(`Pym message with unknown type (${e.data.type}) received`);
        }
      }
    });

    // sending a message back up to the parent from a child
    var guest = PymParticle.registerGuest();
    guest.sendMessage({
      sentinel: "npr",
      type: "broadcast",
      value: "Hello from the guest!"
    });

    // re-broadcasting to all instances from the host page
    window.addEventListener("message", function(e) {
      // only proceed on our specific messages
      if (!e.data.sentinel || e.sentinel.data != "npr") return;
      // broadcast the message back to all guest pages
      var hosts = document.querySelectorAll("pym-particle");
      hosts.forEach(host => host.sendMessage(e.data));
    });

Open questions
--------------

* What additional functions should be made available in the loader library? Do people generally use the navigate or scroll functions?
* How do people typically use these libraries? Should we offer an unpackaged version via CDN, or embrace NPM?
* This library should address confusion around initializing Pym, problems with page margin, and automatically monitoring page height. Are there other Pym v1 weaknesses or edge cases we can address here?
* Should the guest and host communicate using legacy Pym message formats by default? Is it more important on the guest or the host? Or should both only use the old message formats if manually enabled? Does it really matter? Performance probably isn't an issue, honestly. 
* Can we change the name? In retrospect, calling it "Pym" after the most well-known domestic abuser in comics is not a great look, especially in 2019. New names could reference the grow/shrink capabilities or the embedding/serving part.