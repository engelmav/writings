---
title: Taming WebApp Testing Complexity
author: Vincent Engelmann
---

The Page Object "pattern" allows us to clean up tests, enhance logging, reuse utilities, and tame complexity.

## Representing Pages

The page representation consists of a `Page` superclass and any subclass with a name that resembles the page it describes. For example, if you have a login page, you can describe it as:

```python
class Login(Page):
    def __init__():
        Page.__init(self)
        self.username = TextArea(id_="username", "username field")
        self.password = TextArea(id_="passwd", "password field")
    
    def set_username(username):
        self.username.enter_text(username)
        return self
    
    def set_password(pw):
        self.password.enter_text(pw)
        return self
        TextArea(id_="username", "username field")
```

The `Page` superclass has the primary responsibility of ensuring all pages use the same `wdhelper` singleton for all Selenium actions.

```python
class Page(object):
    def __init__():
        self.wdhelper wdhelper
        self.wdhelper.register(self)
```

## Session Consistency Across Objects

The `WdHelper` singleton maintains the WebDriver session across all pages and elements.

## Representing DOM Elements

```python
from selenium.webdriver.remote.webelement import WebElement
from src.poutils import jsutil
from src.poutils import wdhelper


class E(object):
    """

    Represents a generic HTML element.

    """

    def __init__(self, css=None, xpath=None, id_=None, description):
        self.selector_type = None
        self.selector = None
        self.description = description
        if css is None and xpath is None and id_ is None:
            raise Exception("You must specify either css, xpath, or ID selector.")
            ...

    def _wait_for(wait_type: WaitType, timeout: int) -> WebElement:
        ...

    def click(timeout=None, auto_scroll=False):
        self.element = self._wait_for(WaitType.CLICKABLE, timeout)
        if auto_scroll:
            jsutil.auto_scroll()
        self.element.click()

    def get_text(timeout=None):
        self._wait_for(WaitType.PRESENT, timeout)
```

We used inheritance to make intent clearer and keep things maintainable:

```python
class TextArea(E):
    def __init__(*args, **kwargs):
        E.__init__(self, *args, **kwargs)

    def get_text(timeout=None):
        """
        Overrides the E class's `get_text()` method. Extracts
        the text from an HTML TextArea element.
        """
        self.element = self._wait_for(WaitType.PRESENT, timeout)
        return self.element.text 

    def enter_text(text: str, timeout=None) -> 'TextArea':
        self.element = self._wait_for(WaitType.PRESENT, timeout)
        self.element.clear().enter_keys(text)
```

We can re-use ``E``'s methods, implement different behaviors for common actions, and add additional actions appropriate to the element type.
WaitType is an Enum.
