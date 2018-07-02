# mbtest

Python wrapper & utils for the [Mountebank](http://www.mbtest.org/) over the wire test double tool.

Includes [pytest](https://pytest.org) fixture and [PyHamcrest](https://pyhamcrest.readthedocs.io) matchers.

## Basic example

```python
import pytest
import requests
from hamcrest import assert_that, is_
from matchers.response import response_with
from mbtest.matchers import had_request
from mbtest.imposters import Imposter, Predicate, Response, Stub

@pytest.mark.usefixtures("mock_server")
def test_request_to_mock_server(mock_server):
    # Start mock server with required behavior
    imposter = Imposter(Stub(Predicate(path="/test"), Response(body="sausages")), record_requests=True)

    with mock_server(imposter) as server:
        # Make request to mock server
        response = requests.get("{}/test".format(imposter.url))

        assert_that("We got the expected response", response, is_(response_with(status_code=200, body="sausages")))
        assert_that("The mock server recorded the request", server, had_request(path="/test", method="GET"))
```

Needs a pytest fixture, usually defined in a [`conftest.py`](https://docs.pytest.org/en/latest/fixture.html#fixture-function):

```python
import pytest
from mbtest import server

@pytest.fixture(scope="session")
def mock_server(request):
    return server.mock_server(request)
```
