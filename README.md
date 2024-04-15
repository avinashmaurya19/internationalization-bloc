# internationalization-bloc

main.dart

import 'package:appening_new_project/Language/bloc/localization_bloc.dart';
import 'package:appening_new_project/helper/Routes/generated_routes.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

import 'package:flutter_gen/gen_l10n/app_localizations.dart';
import 'package:flutter_localizations/flutter_localizations.dart';

// void main() {
//   runApp(BlocProvider(
//     create: (context) => LocalizationBloc()..add(LoadSavedLocalization()),
//     child: const MyApp(),
//   ));
// }

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<LocalizationBloc, LocalizationState>(
      builder: (context, state) {
        return MaterialApp(
          debugShowCheckedModeBanner: false,
          locale: const Locale('en'),
          // locale: state.locale,
          localizationsDelegates: const [
            AppLocalizations.delegate,
            GlobalMaterialLocalizations.delegate,
            GlobalWidgetsLocalizations.delegate,
            GlobalCupertinoLocalizations.delegate,
          ],
          supportedLocales: AppLocalizations.supportedLocales,
          title: 'Flutter Demo',
          theme: ThemeData(
            useMaterial3: true,
          ),
          initialRoute: '/',
          onGenerateRoute: RouteGenerator().generateRoute,
        );
      },
    );
  }
}



event

part of 'localization_bloc.dart';

@immutable
sealed class LocalizationEvent {}

class LoadLocalization extends LocalizationEvent {
  final Locale locale;
   LoadLocalization(this.locale);
}

class LoadSavedLocalization extends LocalizationEvent {
  
}

state

part of 'localization_bloc.dart';

@immutable
 class LocalizationState {
  final Locale locale;
  const LocalizationState(this.locale);


  //Default state
  // factory LocalizationState.initial() {
  //   return const LocalizationState(Locale('en'));
  // }
 }

// LocalizationState copyWith(Locale locale) {
//   return LocalizationState(locale);
// }

bloc

import 'dart:ui';

import 'package:bloc/bloc.dart';
import 'package:meta/meta.dart';
import 'package:shared_preferences/shared_preferences.dart';

part 'localization_event.dart';
part 'localization_state.dart';

class LocalizationBloc extends Bloc<LocalizationEvent, LocalizationState> {
  // LocalizationBloc() : super(LocalizationState.initial()) {
  LocalizationBloc() : super(const LocalizationState(Locale('en'))) {
    
    on<LoadSavedLocalization>((event, emit) async {
      Locale saveLocale = await getLocale();
      emit(LocalizationState(saveLocale));
    });

    on<LoadLocalization>((event, emit) {
      if (event.locale == state.locale) {
        return;
      }
      saveLocale(event.locale);
      emit(LocalizationState(event.locale));
    });
  }
  saveLocale(Locale locale) async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    prefs.setString('language', locale.languageCode);
  }

  Future<Locale> getLocale() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    String languageCode = prefs.getString('language') ?? 'en';
    // ignore: avoid_print
    print(languageCode);
    return Locale(languageCode);
  }
}

//Language Screen 
  
class LanguageScreen extends StatelessWidget {
  const LanguageScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // var groupValue = 'es';
    var groupValue = context.read<LocalizationBloc>().state.locale.languageCode;
    return BlocConsumer<LocalizationBloc, LocalizationState>(
      listener: (context, state) {
        groupValue = state.locale.languageCode;
      },
      builder: (context, state) {
        return Scaffold(
          appBar: AppBar(
            title: Text(
              AppLocalizations.of(context)!.language,
            ),
          ),
          body: ListView.builder(
              itemCount: languageModel.length,
              itemBuilder: (context, index) {
                var item = languageModel[index];
                return RadioListTile(
                  value: item.languageCode,
                  groupValue: groupValue,
                  title: Text(item.language),
                  // subtitle: Text(item.subLanguage),
                  onChanged: (value) {
                    BlocProvider.of<LocalizationBloc>(context)
                        .add(LoadLocalization(Locale(item.languageCode)));
                  },
                );
              }),
        );
      },
    );
  }
}



popmenuitem

  void showLanguageMenu(BuildContext context) {
    final localizationBloc = BlocProvider.of<LocalizationBloc>(context);

    // Get the current language code to determine the selected value
    String currentLanguageCode = localizationBloc.state.locale.languageCode;

    // Display a pop-up menu with language options
    showMenu<String>(
      context: context,
      position: const RelativeRect.fromLTRB(
          50, 80, 20, 0), // Adjust position as needed
      items: languageModel.map((item) {
        return PopupMenuItem<String>(
          value: item.languageCode,
          child: RadioListTile<String>(
            value: item.languageCode,
            groupValue: currentLanguageCode,
            title: Text(item.language),
            onChanged: (value) {
              // Change language when a new language is selected
              localizationBloc.add(LoadLocalization(Locale(value!)));
              Navigator.pop(context); // Close the pop-up menu
            },
          ),
        );
      }).toList(),
    );
  }
