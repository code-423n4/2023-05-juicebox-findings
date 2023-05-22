# JBXBuybackDelegate is Ownable, but its never used
Consider remove Ownable inheretance since _owner stoage variable set in constructor, but it never read

# In didPay mintedAmount reset to 1 is unnecessary
Consider remove unnecessary reset mintedAmount to 1 since it will be overwriten in then next pay -> payParams callchain 