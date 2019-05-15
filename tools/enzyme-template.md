Create a new template:

![Screenshot_from_2017-08-22_09-51-01](/uploads/494183530aba8257025732ab114a6806/Screenshot_from_2017-08-22_09-51-01.png)
![Screenshot_from_2017-08-22_09-51-17](/uploads/1bd55e43e929c78e02f127cd705a4a8b/Screenshot_from_2017-08-22_09-51-17.png)

Here is the template:
```
// @flow
import React from 'react'
import { shallow } from 'enzyme';
import { shallowToJson } from 'enzyme-to-json';
import sinon from 'sinon';
import $COMPONENT from '../$COMPONENT';

describe("$COMPONENT", () => {
  it("should not be null", () => {
    const onChange = sinon.spy();
    const wrapper = shallow(
      <$COMPONENT
        onChange={onChange} 
      />
    );
    expect(wrapper).not.toBe(null);
  });
});
```

Create a new test by right clicking the ```__tests__``` folder:
![Untitled](/uploads/92a9dbd9287255e032702086af5639f3/Untitled.png)
![Screenshot_from_2017-08-22_10-01-47](/uploads/df31c45a967b56b039dafe8434b520ea/Screenshot_from_2017-08-22_10-01-47.png)