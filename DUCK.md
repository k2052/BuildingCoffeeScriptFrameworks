We should use this as the teaser

class Platypus extends mixOf Otter, Duck

it 'should have fur and a bill', ->
  platypus = new Platypus
  platypus.hasFur().should.equal true
  platypus.hasBill().should.equal true