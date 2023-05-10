pg-anon-dump
=============

Export your PostgreSQL dump file to anonymize. Replace all sensitive data thanks to `faker`. Output to a file that you can easily import with `psql`.

## Usage

Run this command by giving a path to dump file and an output file name (no need to install first thanks to `npx`):

```bash
npx pg-anon-dump in.sql -o dump.sql
```

Output can also be stdout ('-') so you can pipe the output to zip, gz, or to psql:

```bash
npx pg-anon-dump in.sql -o - | psql DATABASE_URL
```

### Specify list of columns to anonymize

Use `--list` option with a comma separated list of column name:

```bash
npx pg-anon-dump in.sql \
  --list=email,firstName,lastName,phone
```

Specifying another list via `--list` replace the default automatically anonymized values:

```csv
email,name,description,address,city,country,phone,comment,birthdate
```

You can also specify the table for a column using the dot notation:

```csv
public.user.email,public.product.description,email,name
```

Alternatively use `--configFile` option to specify a file with a list of column names and optional replacements, one per line:

```bash
npx pg-anon-dump in.sql \
  --configFile /path/to/file
```

#### Customize replacements 

You can also choose which faker function you want to use to replace data (default is `faker.random.word`):

```bash
npx pg-anon-dump in.sql \
  --list=firstName:faker.name.firstName,lastName:faker.name.lastName
```

:point_right: You don't need to specify faker function since the command will try to find correct function via column name.

You can use plain text too for static replacements:
```bash
npx pg-anon-dump in.sql \
  --list=textcol:hello,jsoncol:{},intcol:12
```

You can even use your custom replacements function from your own javascript module.
Here is a simple example to mask all the email.
```bash
npx pg-anon-dump in.sql \
  --extension ./myExtension.js \
  --list=email:extension.maskEmail
```

```javascript
// myExtension.js
module.exports = {
  maskEmail: (email) => {
   const [name, domain] = email.split('@');
   const { length: len } = name;
   const maskedName = name[0] + '...' + name[len - 1];
   const maskedEmail = maskedName + '@' + domain;
   return maskedEmail;
  }
};
```
