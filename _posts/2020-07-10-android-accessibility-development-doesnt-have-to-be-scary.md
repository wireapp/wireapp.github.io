---
layout: post
title:  "Android Accessibility Development Doesn't Have to Be Scary"
author: bradley
description: Android accessibility development can be seen as a daunting task, but it doesn't have to be. Here are some tips to make it less scary. 
categories: [ engineering, accessibility, android ]
image: assets/images/posts/accessibility_article_featured.jpg
featured: true
---

You can also find this article on [Medium](https://medium.com/swlh/android-accessibility-development-doesnt-have-to-be-scary-971cfe713a0e). 

> As developers, we make many assumptions about how users consume our application and that can actually hinder instead of help them. We need to ensure we're
> keeping our flows as simple as possible and make everything accessible to every user in a way that makes sense to them, not us.

In this article, I want to share a few tips that really do make accessibility development less daunting. Apply some of these and you're already on your way to an accessible Android application.

The more complicated the UX flows, the more we need to customise the accessibility UX flows alongside it. This doesn't mean we can't have complicated flows, only that we need to ensure we're considering **all** of our users when developing them.

## Tip #1: Use Native Components

You **should** use or extend native views as much as possible for the purpose they were intended.

The Android framework gives us a plethora of capabilities when it comes to Android components and does a good job of translating these components in an accessible way. The ecosystem relies on us developers to fill in the blanks to tell the OS what our application does, so everyone can have the most consistent experience possible on their device.

If you create a custom component that doesn't extend from the intended view you need, then you lose those accessibility capabilities for that. Even using the wrong view for the wrong job can lose information for an accessibility user. We need to ensure we're as informative as possible in a simple way.

**Example**:

If we use a `TextView`, apply a click listener to it to navigate somewhere else. A non-accessible user may not notice anything different. But for an accessibility user, we lose some key talkback capabilities that come with it.

```
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="Terms and conditions"/>
```

If we have talkback enabled and navigate to this `TextView` the user will hear:

> "Terms and conditions - Double-tap to activate"

But if we use a `Button` (which is the intended component here):

```
<Button
    style="@style/Widget.MaterialComponents.Button.TextButton"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="Terms and conditions"/>
```

> "Terms and conditions button - Double-tap to activate"

These subtle changes make it consistent and provide more information for the accessibility users who are not only using your application, but other apps in the ecosystem.

## Tip #2: Avoid forcing focus on views

Focus **should not** be forced on any component for accessible users without a clear intent or interaction from them.

Accessibility users need to have a consistent flow when navigating; either touch by exploration, or via a switch device (as defined by WCAG 2.0 accessibility guidelines [3.2.1](https://www.w3.org/WAI/WCAG21/Understanding/on-focus.html) and [3.2.3](https://www.w3.org/TR/UNDERSTANDING-WCAG20/consistent-behavior-consistent-locations.html) respectively). If we decide to not only focus the OS accessibility delegate or any component without a clear interaction from the user, then it will create inconsistent behaviour for the user, it can also create bugs from the accessibility side because we're overriding the OS's accessibility event hierarchy;

**Examples**:
* `<requestFocus/>` in your layout's xml
* `view.sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_FOCUSED)`
* `android:accessibilityTraversalBefore` and `android:accessibilityTraversalAfter`

If you still need to request focus for better user experience, you can query the `AccessibilityManager` service to determine if a particular accessibility service is enabled and then only request focus when it **isn't**.

**Example**:

```
fun View.screenReaderFocusable() =
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
        isScreenReaderFocusable = true
    } else {
        ViewCompat.setAccessibilityDelegate(this, object : AccessibilityDelegateCompat() {
            override fun onInitializeAccessibilityNodeInfo(
                host: View,
                info: AccessibilityNodeInfoCompat
            ) {
                info.isScreenReaderFocusable = true
                super.onInitializeAccessibilityNodeInfo(host, info)
            }
        })
    }
```
You can call it like this:

```
private fun requestFocus(view: View) {
    if (!accessibility.isTalkbackEnabled()) {
        view.requestFocus()
    }
}
```

If you want more control of focus when talkback is enabled take a look at android:ScreenReaderFocusable (which was introduced in API 28). If you are using AndroidX, you can also achieve this in a backwards compatible way (API 19 and above) like below:

## Tip #3: Touch Target Size

As a rule of thumb, all touchable components should be at least 48dp in height by 48dp in width so users with dexterity issues can navigate the application easier. You can achieve this in several ways:

* Global `android:minWidth` and `android:minWidth` styling for all separate touchable components using your application theme. Or, apply individual styles to your views in xml.
* Apply padding to your components to make the touch area larger.
* Implement a [TouchDelegate](https://developer.android.com/reference/android/view/TouchDelegate) to expand the touch area to be larger than the bounds of the view.

## Tip #4: Is the view important for accessibility?

Is the view important for a user who may be navigating with switch access or exploration by touch?

When navigating through UI flows, we want the experience to be as seamless as possible. If we have components that purely exist to make the application user experience nicer, but doesn't add too much value for accessibility users then utilise the [importantForAccessibility](https://developer.android.com/reference/android/view/View#attr_android:importantForAccessibility) attribute (API 16 or higher).

**Note**: For components that do not add too much value to accessibility, or exist purely for decoration, you can also set the `contentDescription="@null"` which won't hurt the navigation experience for switch access users.

## Tip #5: Help talkback users navigate faster

Talkback users can swipe up to change the controls for how screen readers navigates the content on the screen. The screen reader can be followed in multiple ways (Headings, Paragraphs, Lines, Words, Characters, Controls, Links). I'll cover examples for headings and links below, you can find out about the rest [here](https://support.google.com/accessibility/android/answer/6006598?hl=en&ref_topic=3529932).

### Headings
The Android OS doesn't know too much about what a heading is in the context of your application, so you need to tell it. This is where the accessibilityHeading attribute comes in.

For those targeting API 28 or above, you can simply add this to your view xml attribute as true. Otherwise, you can override the node information on a new `AccessibilityDelegate` for the view you want to declare a header and set the isHeading value true.

```
fun View.headingForAccessibility() =
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
        isAccessibilityHeading = true
    } else {
        ViewCompat.setAccessibilityDelegate(this, object : AccessibilityDelegateCompat() {
            override fun onInitializeAccessibilityNodeInfo(
                host: View,
                info: AccessibilityNodeInfoCompat
            ) {
                info.isHeading = true
                super.onInitializeAccessibilityNodeInfo(host, info)
            }
        })
    }

```
You can then use it like so:

```
headingTextView.headingForAccessibility()

```

This will now not only read out the text with "heading" at the end, but it'll also allow the user to only gain focus on views which are defined as headings within the application to make navigation much faster when they have headings navigation control turned on.

### Links

Links allows you to navigate [URLSpan](https://developer.android.com/reference/android/text/style/URLSpan) components or TextViews that have been filtered using ][Linkify](https://developer.android.com/reference/android/text/util/Linkify.html) (it'll search the text and find a corresponding link). A link can be defined as a home address, phone number, email address or web URL.

By using the same principles mentioned above around native components - we can create a simple custom `TextView` that will not just set a `minWidth` and `minHeight` to the view (now that we know it's a touchable component) it'll also filter out the text to find a link and apply it to the text.

**Example**:

You would implement it like so:

```
class LinkTextView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyle: Int = 0
) : AppCompatTextView(context, attrs, defStyle) {
    init {
        minWidth = context.resources.getDimension(R.dimen.min_touchable_width).toInt()
        minHeight = context.resources.getDimension(R.dimen.min_touchable_height).toInt()
        Linkify.addLinks(this, Linkify.ALL)
    }
}
```

```
<com.bradley.wilson.accessibility.views.LinkTextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:gravity="center_horizontal"
    android:text="Go to www.google.com" />

```

www.google.com will be hyperlinked. Now when you turn talkback on, swipe up the navigation to links. You can navigate all links on the screen without navigating through every component to find it.

Other key navigation techniques such as controls rely on finding Android components (checkboxes, radio buttons, switches, sliders (seek controls), text fields, and buttons) to navigate to, so if you're using native components or custom components that extend the correct native component, the OS should do all the work for you.

The rest is simply text navigation, as long as you have text to consume on the screen the OS will take care of the rest.

## Tip #6: Information about screen content
Also introduced in API 28 was the ability to set titles to particular areas of a screen, these are referred to as [panes](https://developer.android.com/about/versions/pie/android-9.0#a11y-pane-titles). They can refer to a `ViewGroup` or `Fragment` content.

```
fun View.paneTitleForAccessibility(title: String) =
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
        accessibilityPaneTitle = title
    } else {
        ViewCompat.setAccessibilityDelegate(this, object : AccessibilityDelegateCompat() {
            override fun onInitializeAccessibilityNodeInfo(
                host: View,
                info: AccessibilityNodeInfoCompat
            ) {
                info.paneTitle = title
                super.onInitializeAccessibilityNodeInfo(host, info)
            }
        })
    }
```


It'd look something like this:

```
paneView.paneTitleForAccessibility(getString(R.string.pain_title))
```

**Note**: As accessibility is user facing, always use strings from the strings.xml so you can take advantage of [localisation](https://developer.android.com/guide/topics/resources/localization).

## Tip #7: Testing can be easy!

### Analysis tools

Testing from within our development team (outside of actually turning on the accessibility services ourselves) we can utilise some key tools introduced by the Android accessibility team to make our lives easier:

1. Pre launch report on the Google Play console. This tool takes advantage of firebase labs to run tests on the app and then generates a report which includes accessibility issues found in the scanned areas of your application. You can find more information [here](https://support.google.com/googleplay/android-developer/answer/7002270?hl=en).

2. UI testing - The team over at [Android Accessibility Test Framework](https://github.com/google/Accessibility-Test-Framework-for-Android) have integrated their tools into Roboelectric and Espresso. You can find an [example](https://github.com/BradleyWire/Accessibility-Playground/blob/master/app/src/androidTest/java/com/bradley/wilson/accessibility/core/ActivityTest.kt) of Espresso UI testing in the playground project.

3. [Accessibility Scanner app](https://play.google.com/store/apps/details?id=com.google.android.apps.accessibility.auditor&hl=en) - They have also made it super easy for non-technical testing to take place to verify whether your application is accessible or not by using a pre-existing application that is also built around the Android Accessibility Test Framework. Simply download the app, enable the accessibility scanner in your device accessibility settings and then follow the options you can take in the app.

**Note**: As the time of writing the Android Accessibility Test Framework migration to AndroidX (version 3.1) hasn't been released. So I still utilise the support version of espresso-accessibility to have AccessibilityChecks run in my UI tests.

### Emulator

Many developers won't always be using a real device to test, If you'd like to access key accessibility features on an emulator, make sure you download the Android Accessibility Suite application onto your emulator and then go to the device settings to find the accessibility services you'd like to enable.

> It's harder for someone to use an inaccessible app than it is to make it accessible.

## Conclusion

I hope you've learnt something new. I would really appreciate any feedback; if you have any better ways to attack a problem; please don't hesitate to leave a comment, post a Github issue or raise a pull request to the playground project.

---

I've created a very simple application to demonstrate some of these principles in a [playground project](https://github.com/BradleyWire/Accessibility-Playground). The more articles I write the more I'll update this repository.

---

### Useful resources:

* [Making Android Accessibility Easy (Android Dev Summit '18)](https://www.youtube.com/embed/R2NftUX7rDM)
* [Accessibility - Android developers](https://developer.android.com/guide/topics/ui/accessibility)
* [Material design - Understanding accessibility](https://material.io/design/usability/accessibility.html#understanding-accessibility)