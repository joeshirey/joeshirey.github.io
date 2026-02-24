source "https://rubygems.org"

# Ruby 3.2+ compatibility shim for Jekyll 3.9 / Liquid 4
if RUBY_VERSION >= "3.2"
  [Object, String, Symbol, Array, Hash, Numeric, NilClass, TrueClass, FalseClass].each do |klass|
    klass.class_eval do
      def tainted?; false; end
      def taint; self; end
      def untaint; self; end
      def trust; self; end
      def untrust; self; end
      def untrusted?; false; end
    end
  end
end

gem "jekyll"
gem "jekyll-feed"
gem "jekyll-seo-tag"
gem "csv"
gem "webrick"
gem "bigdecimal"
