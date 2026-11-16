# Dart and Flutter Formatting Examples

Read [common rules](common.md) for length limits, helper reuse, and test exemptions.
Use `dart format` and the project's Dart analyzer configuration.
Use Flutter-specific guidance below for Flutter widgets.

## Production code

The search screen handles input, loading, results, empty collections, errors, and disposal.
The imports under `package:catalog/` are dummy application paths.
`Product` has a constant constructor with `id`, `name`, and `category` fields.
`ProductRepository.search(String query)` returns `Future<List<Product>>`.
Request identifiers prevent older asynchronous completions from replacing newer results.
Both versions retain the same state transitions and lifecycle guards.

### Incorrect

Logical groups run together across the widget and its methods.
Reusable results and card UI are built through helper methods inside the state class.
The product-tile helper occupies one line.

```dart
import 'package:flutter/material.dart';
import 'package:catalog/model/product.dart';
import 'package:catalog/repository/product_repository.dart';
class ProductSearchPage extends StatefulWidget {
  const ProductSearchPage({required this.repository, super.key});
  final ProductRepository repository;
  @override
  State<ProductSearchPage> createState() => _ProductSearchPageState();
}
class _ProductSearchPageState extends State<ProductSearchPage> {
  final _query = TextEditingController();
  List<Product> _products = [];
  String? _error;
  bool _loading = false;
  bool _hasSearched = false;
  int _requestId = 0;
  Future<void> _search() async {
    final query = _query.text.trim();
    if (query.isEmpty) return;
    final requestId = ++_requestId;
    setState(() {
      _loading = true;
      _error = null;
      _hasSearched = true;
    });
    try {
      final products = await widget.repository.search(query);
      if (!mounted || requestId != _requestId) return;
      setState(() {
        _products = products;
      });
    } catch (_) {
      if (!mounted || requestId != _requestId) return;
      setState(() {
        _error = 'Products could not be loaded.';
      });
    } finally {
      if (mounted && requestId == _requestId) {
        setState(() {
          _loading = false;
        });
      }
    }
  }
  @override
  void dispose() {
    _query.dispose();
    super.dispose();
  }
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Product search')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _query,
              decoration: const InputDecoration(labelText: 'Search products'),
              onSubmitted: (_) => _search(),
            ),
            const SizedBox(height: 12),
            FilledButton(
              onPressed: _loading ? null : _search,
              child: const Text('Search'),
            ),
            const SizedBox(height: 16),
            Expanded(
              child: _buildResults(),
            ),
          ],
        ),
      ),
    );
  }
  Widget _buildResults() {
    if (_loading) {
      return const Center(child: CircularProgressIndicator());
    }
    if (_error != null) {
      return Center(child: Text(_error!));
    }
    if (!_hasSearched) {
      return const Center(child: Text('Enter a search term.'));
    }
    if (_products.isEmpty) {
      return const Center(child: Text('No products found.'));
    }
    return ListView.builder(
      itemCount: _products.length,
      itemBuilder: (context, index) => _buildProductTile(_products[index]),
    );
  }
  Widget _buildProductTile(Product product) { return ListTile(title: Text(product.name), subtitle: Text(product.category)); }
}
```

### Correct

Keep input and asynchronous state handling with the page.
Extract reusable UI into widgets and keep domain-independent logic in suitable helpers.
Preserve mounted/request checks and controller disposal during extraction.

The page lives in `lib/product_search_page.dart`.

```dart
import 'package:flutter/material.dart';

import 'package:catalog/model/product.dart';
import 'package:catalog/repository/product_repository.dart';
import 'package:catalog/widgets/product_results.dart';

class ProductSearchPage extends StatefulWidget {
  const ProductSearchPage({required this.repository, super.key});

  final ProductRepository repository;

  @override
  State<ProductSearchPage> createState() => _ProductSearchPageState();
}

class _ProductSearchPageState extends State<ProductSearchPage> {
  final _query = TextEditingController();
  List<Product> _products = [];
  String? _error;
  bool _loading = false;
  bool _hasSearched = false;
  int _requestId = 0;

  Future<void> _search() async {
    final query = _query.text.trim();
    if (query.isEmpty) return;

    final requestId = ++_requestId;
    setState(() {
      _loading = true;
      _error = null;
      _hasSearched = true;
    });

    try {
      final products = await widget.repository.search(query);
      if (!mounted || requestId != _requestId) return;

      setState(() {
        _products = products;
      });
    } catch (_) {
      if (!mounted || requestId != _requestId) return;

      setState(() {
        _error = 'Products could not be loaded.';
      });
    } finally {
      if (mounted && requestId == _requestId) {
        setState(() {
          _loading = false;
        });
      }
    }
  }

  @override
  void dispose() {
    _query.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Product search')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _query,
              decoration: const InputDecoration(labelText: 'Search products'),
              onSubmitted: (_) => _search(),
            ),
            const SizedBox(height: 12),
            FilledButton(
              onPressed: _loading ? null : _search,
              child: const Text('Search'),
            ),
            const SizedBox(height: 16),
            Expanded(
              child: ProductResults(
                loading: _loading,
                hasSearched: _hasSearched,
                error: _error,
                products: _products,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

The results and card widgets live in `lib/widgets/product_results.dart`.

```dart
import 'package:flutter/material.dart';

import 'package:catalog/model/product.dart';

class ProductResults extends StatelessWidget {
  const ProductResults({
    required this.loading,
    required this.hasSearched,
    required this.error,
    required this.products,
    super.key,
  });

  final bool loading;
  final bool hasSearched;
  final String? error;
  final List<Product> products;

  @override
  Widget build(BuildContext context) {
    if (loading) {
      return const Center(child: CircularProgressIndicator());
    }
    if (error != null) {
      return Center(child: Text(error!));
    }
    if (!hasSearched) {
      return const Center(child: Text('Enter a search term.'));
    }
    if (products.isEmpty) {
      return const Center(child: Text('No products found.'));
    }

    return ListView.builder(
      itemCount: products.length,
      itemBuilder: (context, index) => ProductTile(product: products[index]),
    );
  }
}

class ProductTile extends StatelessWidget {
  const ProductTile({required this.product, super.key});

  final Product product;

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text(product.name),
      subtitle: Text(product.category),
    );
  }
}
```

## Widget tests

The fake repository exposes `complete(products)`, `fail(error)`, and `dispose()` for a pending search.
Its `dispose()` releases pending operations and test resources.
Test code uses the same product page and explicit Flutter imports.

### Incorrect

Every test repeats repository setup, cleanup registration, mounting, and search submission.
Logical phases have no empty-line separation.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:catalog/model/product.dart';
import 'package:catalog/product_search_page.dart';
import 'package:catalog/testing/fake_product_repository.dart';

void main() {
  testWidgets('shows search results', (tester) async {
    final repository = FakeProductRepository();
    addTearDown(repository.dispose);
    await tester.pumpWidget(
      MaterialApp(home: ProductSearchPage(repository: repository)),
    );
    await tester.enterText(find.byType(TextField), 'lamp');
    await tester.tap(find.text('Search'));
    await tester.pump();
    expect(find.byType(CircularProgressIndicator), findsOneWidget);
    repository.complete([
      const Product(id: '1', name: 'Desk lamp', category: 'Lighting'),
    ]);
    await tester.pumpAndSettle();
    expect(find.text('Desk lamp'), findsOneWidget);
  });
  testWidgets('shows a search failure', (tester) async {
    final repository = FakeProductRepository();
    addTearDown(repository.dispose);
    await tester.pumpWidget(
      MaterialApp(home: ProductSearchPage(repository: repository)),
    );
    await tester.enterText(find.byType(TextField), 'lamp');
    await tester.tap(find.text('Search'));
    await tester.pump();
    repository.fail(StateError('offline'));
    await tester.pumpAndSettle();
    expect(find.text('Products could not be loaded.'), findsOneWidget);
  });
  testWidgets('shows an empty result', (tester) async {
    final repository = FakeProductRepository();
    addTearDown(repository.dispose);
    await tester.pumpWidget(
      MaterialApp(home: ProductSearchPage(repository: repository)),
    );
    await tester.enterText(find.byType(TextField), 'lamp');
    await tester.tap(find.text('Search'));
    await tester.pump();
    repository.complete([]);
    await tester.pumpAndSettle();
    expect(find.text('No products found.'), findsOneWidget);
  });
  testWidgets('ignores completion after disposal', (tester) async {
    final repository = FakeProductRepository();
    addTearDown(repository.dispose);
    await tester.pumpWidget(
      MaterialApp(home: ProductSearchPage(repository: repository)),
    );
    await tester.enterText(find.byType(TextField), 'lamp');
    await tester.tap(find.text('Search'));
    await tester.pump();
    await tester.pumpWidget(const SizedBox.shrink());
    repository.complete([
      const Product(id: '1', name: 'Desk lamp', category: 'Lighting'),
    ]);
    await tester.pump();
    expect(tester.takeException(), isNull);
  });
}
```

### Correct

Reuse mounting and search helpers while keeping query inputs and expected states visible.
Preserve teardown through `addTearDown` and check late completion after page disposal.
Test files and functions retain formatting and line-length rules while receiving size exemptions.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:catalog/model/product.dart';
import 'package:catalog/product_search_page.dart';
import 'package:catalog/testing/fake_product_repository.dart';

void main() {
  testWidgets('shows search results', (tester) async {
    final repository = await mountProductSearch(tester);

    await submitSearch(tester, 'lamp');

    expect(find.byType(CircularProgressIndicator), findsOneWidget);
    repository.complete([
      const Product(id: '1', name: 'Desk lamp', category: 'Lighting'),
    ]);
    await tester.pumpAndSettle();

    expect(find.text('Desk lamp'), findsOneWidget);
  });

  testWidgets('shows a search failure', (tester) async {
    final repository = await mountProductSearch(tester);

    await submitSearch(tester, 'lamp');

    repository.fail(StateError('offline'));
    await tester.pumpAndSettle();

    expect(find.text('Products could not be loaded.'), findsOneWidget);
  });

  testWidgets('shows an empty result', (tester) async {
    final repository = await mountProductSearch(tester);

    await submitSearch(tester, 'lamp');

    repository.complete([]);
    await tester.pumpAndSettle();

    expect(find.text('No products found.'), findsOneWidget);
  });

  testWidgets('ignores completion after disposal', (tester) async {
    final repository = await mountProductSearch(tester);

    await submitSearch(tester, 'lamp');

    await tester.pumpWidget(const SizedBox.shrink());
    repository.complete([
      const Product(id: '1', name: 'Desk lamp', category: 'Lighting'),
    ]);
    await tester.pump();

    expect(tester.takeException(), isNull);
  });
}

Future<FakeProductRepository> mountProductSearch(WidgetTester tester) async {
  final repository = FakeProductRepository();
  addTearDown(repository.dispose);

  await tester.pumpWidget(
    MaterialApp(home: ProductSearchPage(repository: repository)),
  );

  return repository;
}

Future<void> submitSearch(WidgetTester tester, String query) async {
  await tester.enterText(find.byType(TextField), query);
  await tester.tap(find.text('Search'));
  await tester.pump();
}
```

## Verification

Format complete snippets with `dart format` and analyze temporary application support with the available Dart/Flutter SDK.
Run widget tests for loading, results, empty/error states, and late asynchronous completion.
Also verify that an older pending search cannot replace a newer search result.
Report unavailable SDK or framework checks explicitly.
