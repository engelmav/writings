Title: Rich Data, Messy PDFs
Date: 2015-11-16 19:00
Category: Solutions
Tags: python, parsing

My uncle won a project to implement White Space broadband in Thurman, NY. One of the many interesting challenges was to determine where lots of "off the grid" residences existed. This would help him plan where he could put his whitespace repeaters.

We thought a bit about what information we had access to and determined that the tax rolls of the county would be useful. But we soon realized that they were provided in some semi-structured PDFs. Given the amount of records for each zone, it was clearly a job for a program.


```python
# ...
    def parseFile(self, filename):
        state = 'START'
        recStart = re.compile('\*+') 
        self.f = None
        self.f = open(filename, 'r')
        for line in self.f:
            if state == 'START' and recStart.match(line):
                state = 'CONTENT'
                self.addressLines = []
                self.addressLines.append(self.extractId(line))
                self.addressLines.append(line)
            elif state == 'CONTENT' and  not recStart.match(line):
                self.addressLines.append(line)
            elif state == 'CONTENT' and  recStart.match(line):
                state = 'START'
                self.addressList.append(self.addressLines)
                self.addressLines = []
                self.addressLines.append(self.extractId(line))
                self.addressLines.append(line)
            elif state == 'START' and not recStart.match(line):
                state = 'CONTENT'
                self.addressLines.append(line)
            else:
                print "No valid state for given line."
    def extractId(self,line):
        return self.recIdRgx.match(line).group(1)
```
