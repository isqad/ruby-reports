# Ruby::Reports [![Build Status](https://travis-ci.org/sclinede/ruby-reports.svg)](https://travis-ci.org/sclinede/ruby-reports)

This gem was written for report automation provided by DSL. See [Usage](#usage) for details

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'ruby-reports'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install ruby-reports

## Usage <a id="usage"></a>

Main concept is following:
- You have report class where you define attributes, column names and mapped to them
  keys from source data
- Report has ```query``` method. It defines what object will serve data querying from any source
- (Optional) Report has ```formatter``` method. It defines what object will serve cell value formatting

Example:
```ruby
  class MyNiceReport < Ruby::Reports::CsvReport
    config(
      source: :fetch_data,
      expire_in: 15 * 60,
      encoding: Ruby::Reports::CP1251,
      directory: File.join('/home', 'my_home', 'nice_reports')
    )

    table do
      column 'ID', :id
      column 'User Name', :username
      column 'Email', :email, formatter: :mail_to_link
      column 'Last Seen', :last_seen_date, formatter: :date
    end

    def formatter
      @formatter ||= Formatter.new
    end

    def query
      @query ||= Query.new(self)
    end

    class Formatter
      def mail_to_link(email)
        "mailto:#{email}"
      end

      def date(date)
        Date.parse(date).strftime('%d.%m.%Y')
      end
    end

    class Query < Ruby::Reports::Services::QueryBuilder
      def fetch_data
        [
          {id: 1, username: 'user#1', email: 'user1@reports.org', last_seen_date: '2015/06/06'},
          {id: 2, username: 'user#2', email: 'user2@reports.org', last_seen_date: '2015/02/07'},
          {id: 3, username: 'user#3', email: 'user3@reports.org', last_seen_date: '2015/08/13'}
        ]
      end
    end
  end

  > report = MyNiceReport.build
  # => #<MyNiceReport:0x00000002171a68 ...>
  > report.ready?
  # => true
  > report.filename
  # => '/home/my_home/nice_reports/asdvdsadvasdv.csv'
  > IO.read(report.filename)
  # => "ID;User Name;Email;Last Seen\r\n1;user#1;mailto:user1@reports.org;06.06.2015\r\n
  # 2;user#2;mailto:user2@reports.org;07.02.2015\r\n3;user#3;mailto:user3@reports.org;13.08.2015\r\n"
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/sclinede/ruby-reports.

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

